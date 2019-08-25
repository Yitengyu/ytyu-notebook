在页面初始化时，有时会遇到需要多次请求后端的情况



#### 错误写法

```typescript
async created () {
    this.leaveInfo = JSON.parse(this.$route.query.info)

    this.leaveTypes = await getOptionItems('leave.type')
    this.leave = await getLeave(this.leaveInfo.workflowId) //在上一条语句触发的请求返回之前不会执行
    this.applyUserInfo = await getUserById(this.leaveInfo.employeeId) //在上一条语句触发的请求返回之前不会执行
}
```



#### await写法

```typescript
// file: AppPivotEHRWeb/src/views/leave/LeaveClaim/index.tsx 
// author: slipkinem
async created () {
    this.leaveInfo = JSON.parse(this.$route.query.info)

    let result = await Promise.all([
      getOptionItems('leave.type'),
      getLeave(this.leaveInfo.workflowId),
      getUserById(this.leaveInfo.employeeId),
    ])

    this.leaveTypes = result[0]
    this.leave = result[1]
    this.applyUserInfo = result[2]
}
```



#### 解构写法

```typescript
// file: AppPivotEHRWeb/src/views/profile/ProfileEducationView/index.tsx
// author: luobinghang
created () {
    Promise.all([
        getDegrees(),
        getLectures()
    ]).then((values) => {
        [
            this.degrees,
            this.lectureType
        ] = values
    })
}
```



#### 解构 + await（推荐）

可以看出，以上两种写法是可以结合的，结果可能更美观。

```typescript
async created () {
  [
    this.degrees,
    this.lectureType
  ] = await Promise.all([
    getDegrees(),
    getLectures()
  ])
}
```

