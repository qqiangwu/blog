# 权限管理
+ ACL: Access Control List，访问控制列表，是比较流行的设计方式。通过吧用户和权限挂钩来实现。
+ RBAC：Role Based Access Control，角色访问控制系统，是另一个实现思路。提炼出角色对象，把用户和角色绑定，角色来对应权限，角色和权限没有直接关联，对复杂的系统来说，更加容易管理。