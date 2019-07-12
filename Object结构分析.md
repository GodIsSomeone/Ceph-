### 数据结构分析

#### RGWDataAccess

使用嵌套类的形式，内嵌Bucket和Object两个类  
RGWDataAccess和Bucket互为友元，相互之间可以进行访问。  
RGWDataAccess不能访问Object，Object可以访问RGWDataAccess。  
友元不具备传递性。  

```
class RGWDataAccess
{
  RGWRados *store;
  std::unique_ptr<RGWSysObjectCtx> sysobj_ctx;

public:
  RGWDataAccess(RGWRados *_store);

  class Object;
  class Bucket;

  int get_bucket(const string& tenant,
		 const string name,
		 const string bucket_id,
		 BucketRef *bucket) {
    bucket->reset(new Bucket(this, tenant, name, bucket_id));
    return (*bucket)->init();
  }

  int get_bucket(const RGWBucketInfo& bucket_info,
		 const map<string, bufferlist>& attrs,
		 BucketRef *bucket) {
    bucket->reset(new Bucket(this));
    return (*bucket)->init(bucket_info, attrs);
  }

  using BucketRef = std::shared_ptr<Bucket>;
  using ObjectRef = std::shared_ptr<Object>;

  class Bucket : public enable_shared_from_this<Bucket>
  {
    friend class RGWDataAccess; //互为友元
    friend class Object;        //互为友元

    RGWDataAccess *sd{nullptr};
    RGWBucketInfo bucket_info;   //桶信息
    string tenant;
    string name;
    string bucket_id;
    ceph::real_time mtime;
    map<std::string, bufferlist> attrs;

    RGWAccessControlPolicy policy;    //权限
    int finish_init();

    Bucket()
    int init();
    int init(const RGWBucketInfo& _bucket_info, const map<string, bufferlist>& _attrs);
  public:
    int get_object(const rgw_obj_key& key, ObjectRef *obj);

  };


  class Object
  {
    RGWDataAccess *sd{nullptr};
    BucketRef bucket;
    rgw_obj_key key;

    ceph::real_time mtime;
    string etag;
    std::optional<uint64_t> olh_epoch;
    ceph::real_time delete_at;
    std::optional<string> user_data;

    std::optional<bufferlist> aclbl;

    Object(){}
  public:
    int put(bufferlist& data, map<string, bufferlist>& attrs, const DoutPrefixProvider *dpp); /* might modify attrs */

    void set_mtime(const ceph::real_time& _mtime) {
      mtime = _mtime;
    }

    void set_etag(const string& _etag) {
      etag = _etag;
    }

    void set_olh_epoch(uint64_t epoch) {
      olh_epoch = epoch;
    }

    void set_delete_at(ceph::real_time _delete_at) {
      delete_at = _delete_at;
    }

    void set_user_data(const string& _user_data) {
      user_data = _user_data;
    }

    void set_policy(const RGWAccessControlPolicy& policy);

    friend class Bucket;       //互为友元
  };

  friend class Bucket;
  friend class Object;
};
```

#### 

```
struct rgw_obj_key {
  string name;
  string instance;
  string ns;

  rgw_obj_key() {}
  // cppcheck-suppress noExplicitConstructor
  rgw_obj_key(const string& n) : name(n) {}
  rgw_obj_key(const string& n, const string& i) : name(n), instance(i) {}
  rgw_obj_key(const string& n, const string& i, const string& _ns) : name(n), instance(i), ns(_ns) {}

  rgw_obj_key(const rgw_obj_index_key& k) {
    parse_index_key(k.name, &name, &ns);
    instance = k.instance;
  }

  static void parse_index_key(const string& key, string *name, string *ns) {
    if (key[0] != '_') {
      *name = key;
      ns->clear();
      return;
    }
    if (key[1] == '_') {
      *name = key.substr(1);
      ns->clear();
      return;
    }
    ssize_t pos = key.find('_', 1);
    if (pos < 0) {
      /* shouldn't happen, just use key */
      *name = key;
      ns->clear();
      return;
    }

    *name = key.substr(pos + 1);
    *ns = key.substr(1, pos -1);
  }

  void set(const string& n) {
    name = n;
    instance.clear();
    ns.clear();
  }

  void set(const string& n, const string& i) {
    name = n;
    instance = i;
    ns.clear();
  }

  void set(const string& n, const string& i, const string& _ns) {
    name = n;
    instance = i;
    ns = _ns;
  }

  bool set(const rgw_obj_index_key& index_key) {
    if (!parse_raw_oid(index_key.name, this)) {
      return false;
    }
    instance = index_key.instance;
    return true;
  }

  void set_instance(const string& i) {
    instance = i;
  }

  const string& get_instance() const {
    return instance;
  }

  string get_index_key_name() const {
    if (ns.empty()) {
      if (name.size() < 1 || name[0] != '_') {
        return name;
      }
      return string("_") + name;
    };

    char buf[ns.size() + 16];
    snprintf(buf, sizeof(buf), "_%s_", ns.c_str());
    return string(buf) + name;
  };

  void get_index_key(rgw_obj_index_key *key) const {
    key->name = get_index_key_name();
    key->instance = instance;
  }

  string get_loc() const {
    /*
     * For backward compatibility. Older versions used to have object locator on all objects,
     * however, the name was the effective object locator. This had the same effect as not
     * having object locator at all for most objects but the ones that started with underscore as
     * these were escaped.
     */
    if (name[0] == '_' && ns.empty()) {
      return name;
    }

    return string();
  }

  bool empty() const {
    return name.empty();
  }

  bool have_null_instance() const {
    return instance == "null";
  }

  bool have_instance() const {
    return !instance.empty();
  }

  bool need_to_encode_instance() const {
    return have_instance() && !have_null_instance();
  }

  string get_oid() const {
    if (ns.empty() && !need_to_encode_instance()) {
      if (name.size() < 1 || name[0] != '_') {
        return name;
      }
      return string("_") + name;
    }

    string oid = "_";
    oid.append(ns);
    if (need_to_encode_instance()) {
      oid.append(string(":") + instance);
    }
    oid.append("_");
    oid.append(name);
    return oid;
  }

  bool operator==(const rgw_obj_key& k) const {
    return (name.compare(k.name) == 0) &&
           (instance.compare(k.instance) == 0);
  }

  bool operator<(const rgw_obj_key& k) const {
    int r = name.compare(k.name);
    if (r == 0) {
      r = instance.compare(k.instance);
    }
    return (r < 0);
  }

  bool operator<=(const rgw_obj_key& k) const {
    return !(k < *this);
  }

  static void parse_ns_field(string& ns, string& instance) {
    int pos = ns.find(':');
    if (pos >= 0) {
      instance = ns.substr(pos + 1);
      ns = ns.substr(0, pos);
    } else {
      instance.clear();
    }
  }

  // takes an oid and parses out the namespace (ns), name, and
  // instance
  static bool parse_raw_oid(const string& oid, rgw_obj_key *key) {
    key->instance.clear();
    key->ns.clear();
    if (oid[0] != '_') {
      key->name = oid;
      return true;
    }

    if (oid.size() >= 2 && oid[1] == '_') {
      key->name = oid.substr(1);
      return true;
    }

    if (oid.size() < 3) // for namespace, min size would be 3: _x_
      return false;

    size_t pos = oid.find('_', 2); // oid must match ^_[^_].+$
    if (pos == string::npos)
      return false;

    key->ns = oid.substr(1, pos - 1);
    parse_ns_field(key->ns, key->instance);

    key->name = oid.substr(pos + 1);
    return true;
  }

  /**
   * Translate a namespace-mangled object name to the user-facing name
   * existing in the given namespace.
   *
   * If the object is part of the given namespace, it returns true
   * and cuts down the name to the unmangled version. If it is not
   * part of the given namespace, it returns false.
   */
  static bool oid_to_key_in_ns(const string& oid, rgw_obj_key *key, const string& ns) {
    bool ret = parse_raw_oid(oid, key);
    if (!ret) {
      return ret;
    }

    return (ns == key->ns);
  }

  /**
   * Given a mangled object name and an empty namespace string, this
   * function extracts the namespace into the string and sets the object
   * name to be the unmangled version.
   *
   * It returns true after successfully doing so, or
   * false if it fails.
   */
  static bool strip_namespace_from_name(string& name, string& ns, string& instance) {
    ns.clear();
    instance.clear();
    if (name[0] != '_') {
      return true;
    }

    size_t pos = name.find('_', 1);
    if (pos == string::npos) {
      return false;
    }

    if (name[1] == '_') {
      name = name.substr(1);
      return true;
    }

    size_t period_pos = name.find('.');
    if (period_pos < pos) {
      return false;
    }

    ns = name.substr(1, pos-1);
    name = name.substr(pos+1, string::npos);

    parse_ns_field(ns, instance);
    return true;
  }

  string to_str() const {
    if (instance.empty()) {
      return name;
    }
    char buf[name.size() + instance.size() + 16];
    snprintf(buf, sizeof(buf), "%s[%s]", name.c_str(), instance.c_str());
    return buf;
  }
};
```

### 调用流程

1. 变量声明

```
std::string bucket_name, pool_name, object;
string object_version;
int64_t max_objects = -1; //对象的最大size
int64_t max_size = -1;    //桶的最大size
bool have_max_objects = false; //对象的最多个数
```

2. 变量赋值

```
    else if (ceph_argparse_witharg(args, i, &val, "-o", "--object", (char*)NULL)) {
      object = val;
    } else if (ceph_argparse_witharg(args, i, &val, "--object-version", (char*)NULL)) {
      object_version = val;
    }
    else if (ceph_argparse_witharg(args, i, &val, "--max-objects", (char*)NULL)) 
    {
      max_objects = (int64_t)strict_strtoll(val.c_str(), 10, &err);
      if (!err.empty()) {
        cerr << "ERROR: failed to parse max objects: " << err << std::endl;
        return EINVAL;
      }
      have_max_objects = true;
    }
```

3. 函数调用

```
  if (opt_cmd == OPT_OBJECT_PUT) {           /* 对象的操作 */
    if (bucket_name.empty()) {
      cerr << "ERROR: bucket not specified" << std::endl;
      return EINVAL;
    }
    if (object.empty()) {
      cerr << "ERROR: object not specified" << std::endl;
      return EINVAL;
    }

    RGWDataAccess data_access(store);
    rgw_obj_key key(object, object_version);

    RGWDataAccess::BucketRef b;
    RGWDataAccess::ObjectRef obj;

    /* 获取bucket */
    int ret = data_access.get_bucket(tenant, bucket_name, bucket_id, &b);
    if (ret < 0) {
      cerr << "ERROR: failed to init bucket: " << cpp_strerror(-ret) << std::endl;
      return -ret;
    }

    ret = b->get_object(key, &obj);
    if (ret < 0) {
      cerr << "ERROR: failed to get object: " << cpp_strerror(-ret) << std::endl;
      return -ret;
    }

    bufferlist bl;
    ret = read_input(infile, bl);
    if (ret < 0) {
      cerr << "ERROR: failed to read input: " << cpp_strerror(-ret) << std::endl;
    }

    map<string, bufferlist> attrs;
    ret = obj->put(bl, attrs, dpp());
    if (ret < 0) {
      cerr << "ERROR: put object returned error: " << cpp_strerror(-ret) << std::endl;
    }
  }
```

4. 接口实现

```
int RGWDataAccess::Object::put(bufferlist& data,
			       map<string, bufferlist>& attrs, const DoutPrefixProvider *dpp)
{
  RGWRados *store = sd->store;
  CephContext *cct = store->ctx();

  string tag;
  append_rand_alpha(cct, tag, tag, 32);

  RGWBucketInfo& bucket_info = bucket->bucket_info;

  rgw::BlockingAioThrottle aio(store->ctx()->_conf->rgw_put_obj_min_window_size);

  RGWObjectCtx obj_ctx(store);
  rgw_obj obj(bucket_info.bucket, key);

  auto& owner = bucket->policy.get_owner();

  string req_id = store->svc.zone_utils->unique_id(store->get_new_req_id());

  using namespace rgw::putobj;
  AtomicObjectProcessor processor(&aio, store, bucket_info, nullptr,
                                  owner.get_id(), obj_ctx, obj, olh_epoch,
                                  req_id, dpp, null_yield);

  int ret = processor.prepare();
  if (ret < 0)
    return ret;

  DataProcessor *filter = &processor;

  CompressorRef plugin;
  boost::optional<RGWPutObj_Compress> compressor;

  const auto& compression_type = store->svc.zone->get_zone_params().get_compression_type(bucket_info.placement_rule);
  if (compression_type != "none") {
    plugin = Compressor::create(store->ctx(), compression_type);
    if (!plugin) {
      ldout(store->ctx(), 1) << "Cannot load plugin for compression type "
        << compression_type << dendl;
    } else {
      compressor.emplace(store->ctx(), plugin, filter);
      filter = &*compressor;
    }
  }

  off_t ofs = 0;
  auto obj_size = data.length();

  RGWMD5Etag etag_calc;

  do {
    size_t read_len = std::min(data.length(), (unsigned int)cct->_conf->rgw_max_chunk_size);

    bufferlist bl;

    data.splice(0, read_len, &bl);
    etag_calc.update(bl);

    ret = filter->process(std::move(bl), ofs);
    if (ret < 0)
      return ret;

    ofs += read_len;
  } while (data.length() > 0);

  ret = filter->process({}, ofs);
  if (ret < 0) {
    return ret;
  }
  bool has_etag_attr = false;
  auto iter = attrs.find(RGW_ATTR_ETAG);
  if (iter != attrs.end()) {
    bufferlist& bl = iter->second;
    etag = bl.to_str();
    has_etag_attr = true;
  }

  if (!aclbl) {
    RGWAccessControlPolicy_S3 policy(cct);

    policy.create_canned(bucket->policy.get_owner(), bucket->policy.get_owner(), string()); /* default private policy */

    policy.encode(aclbl.emplace());
  }

  if (etag.empty()) {
    etag_calc.finish(&etag);
  }

  if (!has_etag_attr) {
    bufferlist etagbl;
    etagbl.append(etag);
    attrs[RGW_ATTR_ETAG] = etagbl;
  }
  attrs[RGW_ATTR_ACL] = *aclbl;

  string *puser_data = nullptr;
  if (user_data) {
    puser_data = &(*user_data);
  }

  return processor.complete(obj_size, etag,
			    &mtime, mtime,
			    attrs, delete_at,
                            nullptr, nullptr,
                            puser_data,
                            nullptr, nullptr);
}
```

5. 内部实现
