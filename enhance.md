### 1、使用对象字面量

#### 对象字面量可以帮助我们的代码更具有可读性。假设想根据角色显示三种类型的用户，不能使用三元，可以试试使用对象字面量

❌ Bad

```
const { role } = user;

switch (role) {
  case ADMIN:
    return <AdminUser />;
  case EMPLOYEE:
    return <EmployeeUser />;
  case USER:
    return <NormalUser />;
}
```

⭕️ Good

```
const { role } = user;

const components = {
  ADMIN: AdminUser,
  EMPLOYEE: EmployeeUser,
  USER: NormalUser,
};

const Component = components[role];

return <Component />;
```
