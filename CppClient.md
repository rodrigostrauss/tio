Tio C++ client lib tries to emulate STL container whenever possible. If you don't use iterators to change container item values, you can just replace `std::list<std::string>` with `tio::containers::list<std::string>` on your code and your data will on Tio. Now clients can subscribe to container changes.

```
tio::Connection cn;
tio::containers::list<string> l;

cn.Connect("127.0.0.1", 6666);

//
// We need to "connect" the tio::containers::list to a containers
// on a Tio Server. After this it's like using a local container
//
l.create(&cn, "test_list", "volatile_list");
l.clear();

l.push_back("0");
assert(l[0] == "0");

l.push_back("1");
assert(l[l.size()-1] == "1");

l[0] = "xpto";
assert(l[0] == "xpto");

assert(l.pop_back() == "1");
assert(l.pop_front() == "xpto");

//
// Using a map container
//

tio::containers::map<string, string> m;

m.create(&cn, "test_map", "volatile_map");

m["a"] = "b";
assert(m["a"] == "b");

```

You just need to pay attention to the fact that every manipulation or data read from the container generates a round trip to the server. The code `l[l.size()-1] == "1"` will send two commands to the server: `get_count` to get the container size and `set` the last record to "1".

### Serializing structs to Tio ###
You can add struct to a Tio container, as long as you provider the serialize and deserialize functions.

```
namespace XPTO
{
  struct TEST_STRUCT
  {
    string s;
    int i;
  };

  void ToTioData(const TEST_STRUCT& value, TIO_DATA* tiodata)
  {
    std::stringstream str;

    str << value.s << ";" << value.i;

    tiodata_set_string(tiodata, str.str().c_str());
  }
}


void test_structs()
{
  tio::Connection cn;

  cn.Connect("127.0.0.1", 6666);

  tio::containers::list<XPTO::TEST_STRUCT> l;
  XPTO::TEST_STRUCT t;

  l.create(&cn, "xpto", "volatile_list");

  t.i = 10;
  t.s = "test";

  l.push_back(t);

}
```

`TIO_DATA` and `tiodata_set_string` are found on the C client API.