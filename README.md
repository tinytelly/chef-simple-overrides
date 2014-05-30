chef-simple-overrides
=====================
Simple step by step of how chef overrides work.  This shows 3 recipes where one overrides an attribute of the other.
Can be used where you have a need for a common set of attributes, then a child set of overrides.
E.g. a Car is common, BMW is the child, 2014 BMW's is the child of BMW.

Overrides in Chef
========

1) Create the 3 recipes
----
```
knife cookbook create test_common
knife cookbook create test_client_x
knife cookbook create test_environment_x
```

2) Add the common attributes
----
```
vi cookbooks/test_common/attributes/default.rb
```
```
default['test_common']['version'] = 'version_1_common'
default['test_common']['client'] = 'client_common'
default['test_common']['os'] = 'os_windows_common'
```
3)Add the common recipe
----
```
vi cookbooks/test_common/recipes/default.rb
```
```
execute 'echo the path attribute' do
command "echo #{node['test_common']['version']}"
end
```
4)Added the client recipe
----
```
vi cookbooks/test_client_x/recipes/default.rb
```

```
include_recipe "test_common"

node.override['test_common']['client'] = 'client_x'
execute 'echo the path attribute' do
command "echo #{node['test_common']['version']} #{node['test_common']['client']}"
end
```

5) Add the environment recipe
----
```
vi cookbooks/test_environment_x/recipes/default.rb
```
```
include_recipe "test_common"
include_recipe "test_client_x"

node.override['test_common']['os'] = 'os_linux_common'
execute 'echo the path attribute' do
command "echo #{node['test_common']['version']}  #{node['test_common']['client']}  #{node['test_common']['os']}"
end
```
6) Upload the recipes 
----
```
knife cookbook upload test_common
knife cookbook upload test_client_x
knife cookbook upload test_environment_x
```
7) Add the recipes to the run list
----
```
knife node run_list add module2 recipe[test_common]
knife node run_list add module2 recipe[test_client_x]
knife node run_list add module2 recipe[test_environment_x]
```
8) Run the cookbook on the node 
----
```
chef-client 
```
```
Recipe: test_common::default
  * execute[echo the path attribute] action run
    - execute echo version_1_common

Recipe: test_client_x::default
  * execute[echo the path attribute] action run
    - execute echo version_1_common client_x

Recipe: test_environment_x::default
  * execute[echo the path attribute] action run
    - execute echo version_1_common  client_x  os_linux_common
```
