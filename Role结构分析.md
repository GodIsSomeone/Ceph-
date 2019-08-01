### Role简介

A role is similar to a user and has permission policies attached to it, that determine what a role can or can not do. A role can be assumed by any identity that needs it. If a user assumes a role, a set of dynamically created temporary credentials are returned to the user. A role can be used to delegate access to users, applications, services that do not have permissions to access some s3 resources.  

角色和用户类似，并且有许可权限，这些权限决定了角色什么能做，什么不能做。角色可以被假想为任何需要的认证，如果用户担任角色,则一组动态创建的临时凭据将返回给用户。角色可用于将访问权限委派给没有权限访问某些 s3 资源的用户、应用程序和服务。  
[Role简介](http://docs.ceph.com/docs/master/radosgw/role/)

### Role的数据结构

```
  class RGWRole
  {
    using string = std::string;
    static const string role_name_oid_prefix;
    static const string role_oid_prefix;
    static const string role_path_oid_prefix;
    static const string role_arn_prefix;
    static constexpr int MAX_ROLE_NAME_LEN = 64;
    static constexpr int MAX_PATH_NAME_LEN = 512;
    static constexpr uint64_t SESSION_DURATION_MIN = 3600; // in seconds
    static constexpr uint64_t SESSION_DURATION_MAX = 43200; // in seconds

    CephContext *cct;
    RGWRados *store;
    string id;
    string name;
    string path;
    string arn;
    string creation_date;
    string trust_policy;
    map<string, string> perm_policy_map;
    string tenant;
    uint64_t max_session_duration;

    int store_info(bool exclusive);
    int store_name(bool exclusive);
    int store_path(bool exclusive);
    int read_id(const string& role_name, const string& tenant, string& role_id);
    int read_name();
    int read_info();
    void set_id(const string& id) { this->id = id; }
    bool validate_input();
    void extract_name_tenant(const std::string& str);

  public:
    RGWRole();
    const string& get_id() const { return id; }
    const string& get_name() const { return name; }
    const string& get_path() const { return path; }
    const string& get_create_date() const { return creation_date; }
    const string& get_assume_role_policy() const { return trust_policy;}
    const uint64_t& get_max_session_duration() const { return max_session_duration; }

    int create(bool exclusive);
    int delete_obj();
    int get();
    int get_by_id();
    int update();
  };
```
