# Change Login Password in Azkaban

The default login user and password of `Azkaban` is `azkaban` and `azkaban`. The default password should be changed.

## Change Login Password

Enter Azkaban web server host via SSH, and move to the Azakban web server directory.

```agsl
sudo su - azkaban;
cd /usr/lib/azkaban/web;
```

There is a user management file `conf/azkaban-users.xml`.

```agsl
<azkaban-users>
  <user groups="azkaban" password="azkaban" roles="admin" username="azkaban"/>
  <user password="metrics" roles="metrics" username="metrics"/>

  <role name="admin" permissions="ADMIN"/>
  <role name="metrics" permissions="METRICS"/>
</azkaban-users>
```

The value of `password` in the following line needs to be modified.

```agsl
<user groups="azkaban" password="azkaban" roles="admin" username="azkaban"/>
```

After modifying `conf/azkaban-users.xml`, restart `Azkaban` in Chango Admin UI.