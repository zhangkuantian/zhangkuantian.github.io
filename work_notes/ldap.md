# datahub-project 通过LDAP登陆
&nbsp;&nbsp;&nbsp;&nbsp;最近在调研datahub-project，我们希望使用统一的LDAP登陆，之前没弄过LDAP，调试了很久，才终于搞定。
### 调试步骤如下
- 生成jaas.conf文件
- 将jaas.conf 挂载到datahub-frontend的/datahub-frontend/conf目录下或者重新打DockerImage，将jaas.conf拷贝到datahub-frontend的/datahub-frontend/conf目录下， 我们因为做了一些修改，所以就直接在重新打image时，拷贝过去了

### jaas 配置如下
```config
WHZ-Authentication {
com.sun.security.auth.module.LdapLoginModule sufficient
java.naming.security.authentication="simple"
userProvider="ldap://xxx.xxx.xxx.xxx:389/{baseDn}"
userFilter="(&(objectClass=person)(sAMAccountName={USERNAME}))"
java.naming.security.principal="{service_account}"
java.naming.security.credentials="{service_password}"
useSSL=false
debug=true;
};
```
