## policy分析

### bucket policy 
  
  [bucketpolicy](https://docs.ceph.com/docs/master/radosgw/bucketpolicy/)

  We use the RGW ‘tenant’ identifier in place of the Amazon twelve-digit account ID. In the future we may allow you to assign an account ID to a tenant, but for now if you want to use policies between AWS S3 and RGW S3 you will have to use the Amazon account ID as the tenant ID when creating users.  

  每一个tenant独享一个空间的桶。结构类似于：tenant:bucket。  
  Under AWS, all tenants share a single namespace. RGW gives every tenant its own namespace of buckets. There may be an option to enable an AWS-like ‘flat’ bucket namespace in future versions. At present, to access a bucket belonging to another tenant, address it as “tenant:bucket” in the S3 request.  

  桶之间的policy可以在账户之间转移，但是不支持用户、角色、组许可，account可以直接赋值权限给独立的用户。account的桶权限，相当于account下的所有user的桶权限。
  In AWS, a bucket policy can grant access to another account, and that account owner can then grant access to individual users with user permissions. Since we do not yet support user, role, and group permissions, account owners will currently need to grant access directly to individual users, and granting an entire account access to a bucket grants access to all users in that account.  

  桶policy不支持string插值（即相互之间的推导）。  
  Bucket policies do not yet support string interpolation.  

  We support certain s3 condition keys for bucket and object requests.  

#### RGWAccessControlPolicy

```
class RGWAccessControlList
{
protected:
  CephContext *cct;
  /* FIXME: in the feature we should consider switching to uint32_t also
   * in data structures. */
  map<string, int> acl_user_map;
  map<uint32_t, int> acl_group_map;
  list<ACLReferer> referer_list;
  multimap<string, ACLGrant> grant_map;
  void _add_grant(ACLGrant *grant);
public:
  explicit RGWAccessControlList(CephContext *_cct) : cct(_cct) {}
  RGWAccessControlList() : cct(NULL) {}

  void set_ctx(CephContext *ctx) {
    cct = ctx;
  }

  virtual ~RGWAccessControlList() {}

  uint32_t get_perm(const DoutPrefixProvider* dpp,
                    const rgw::auth::Identity& auth_identity,
                    uint32_t perm_mask);
  uint32_t get_group_perm(ACLGroupTypeEnum group, uint32_t perm_mask);
  uint32_t get_referer_perm(uint32_t current_perm,
                            std::string http_referer,
                            uint32_t perm_mask);
  static void generate_test_instances(list<RGWAccessControlList*>& o);

  void add_grant(ACLGrant *grant);

  multimap<string, ACLGrant>& get_grant_map() { return grant_map; }
  const multimap<string, ACLGrant>& get_grant_map() const { return grant_map; }

  void create_default(const rgw_user& id, string name) {
    acl_user_map.clear();
    acl_group_map.clear();
    referer_list.clear();

    ACLGrant grant;
    grant.set_canon(id, name, RGW_PERM_FULL_CONTROL);
    add_grant(&grant);
  }
};

class RGWAccessControlPolicy
{
protected:
  CephContext *cct;
  RGWAccessControlList acl;
  ACLOwner owner;

public:
  explicit RGWAccessControlPolicy(CephContext *_cct) : cct(_cct), acl(_cct) {}
  RGWAccessControlPolicy() : cct(NULL), acl(NULL) {}
  virtual ~RGWAccessControlPolicy() {}

  void set_ctx(CephContext *ctx) {
    cct = ctx;
    acl.set_ctx(ctx);
  }

  uint32_t get_perm(const DoutPrefixProvider* dpp,
                    const rgw::auth::Identity& auth_identity,
                    uint32_t perm_mask,
                    const char * http_referer);
  uint32_t get_group_perm(ACLGroupTypeEnum group, uint32_t perm_mask);
  bool verify_permission(const DoutPrefixProvider* dpp,
                         const rgw::auth::Identity& auth_identity,
                         uint32_t user_perm_mask,
                         uint32_t perm,
                         const char * http_referer = nullptr);

  void set_owner(ACLOwner& o) { owner = o; }
  ACLOwner& get_owner() {
    return owner;
  }

  void create_default(const rgw_user& id, string& name) {
    acl.create_default(id, name);
    owner.set_id(id);
    owner.set_name(name);
  }
  RGWAccessControlList& get_acl() {
    return acl;
  }
  const RGWAccessControlList& get_acl() const {
    return acl;
  }

  virtual bool compare_group_name(string& id, ACLGroupTypeEnum group) { return false; }
};
```

#### ACLOwner

```
    class ACLOwner
    {
    protected:
        rgw_user id;
        string display_name;
    public:
        void dump(Formatter *f) const;
        void decode_json(JSONObj *obj);
        static void generate_test_instances(list<ACLOwner*>& o);
        void set_id(const rgw_user& _id) { id = _id; }
        void set_name(const string& name) { display_name = name; }

        rgw_user& get_id() { return id; }
        const rgw_user& get_id() const { return id; }
        string& get_display_name() { return display_name; }
    };
```
