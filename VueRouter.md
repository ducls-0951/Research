# Getting Started
   
   Tạo một SPA bằng Vue + Vue Router rất đơn giản. Vue phụ trách việc tạo ra các components, Vue Router phụ trách việc điều hướng các components.

### HTML

```
<script src="https://unpkg.com/vue/dist/vue.js"></script>
<script src="https://unpkg.com/vue-router/dist/vue-router.js"></script>

<div id="app">
  <h1>Hello App!</h1>
  <p>
    <!-- sử dụng router-link để điều hướng. -->
    <!-- truyền đường link vào thuộc `to`. -->
    <!-- `<router-link>` khi render sẽ được chuyển thành thẻ `a` -->
    <router-link to="/foo">Go to Foo</router-link>
    <router-link to="/bar">Go to Bar</router-link>
  </p>
  <!-- route outlet -->
  <!-- component được router điều hướng tới sẽ được render trong `<router-view>` -->
  <router-view></router-view>
</div>
```

### Javascript
    
```
// 1. Define Components
const Foo = { template: '<div>foo</div>' }
const Bar = { template: '<div>bar</div>' }

// 2. Define routes
// Mỗi route ánh xạ tới một component.
const routes = [
  { path: '/foo', component: Foo },
  { path: '/bar', component: Bar }
]

// 3. Khởi tạo một router instance và truyền tham số `routes`
const router = new VueRouter({
  routes
})

// 4. Khởi tạo app
const app = new Vue({
    el: "#app",
    router
})
```

# Dynamic Route

Vue Router có cơ chế truyền tham số động vào routes. Tham số truyền vào được bắt đầu bằng ```:```.

```
const User = {
  template: '<div>User</div>'
}

const router = new VueRouter({
  routes: [
    // tham số truyền vào bắt đầu bằng dấu ```:```. Ở đây bắt là ```:id```.
    { path: '/user/:id', component: User }
  ]
})
```

Để lấy được tham số truyền vào trên routes, sử dụng: ```this.$route.params``` hoặc là ```$route.params```. Ở ví dụ trên, muốn lấy trá trị ```id```, làm như sau: ```this.$route.params.id``` hoặc ```$route.params.id```.

```
const User = {
  template: '<div>User {{ $route.params.id }}</div>'
}
```

# Nested Routes

Một UI của ứng dụng thường bao gồm nhiều components lồng nhau. Mỗi component lại ứng với một route.

```
/user/foo/profile                     /user/foo/posts
+------------------+                  +-----------------+
| User             |                  | User            |
| +--------------+ |                  | +-------------+ |
| | Profile      | |  +------------>  | | Posts       | |
| |              | |                  | |             | |
| +--------------+ |                  | +-------------+ |
+------------------+                  +-----------------+
```

```
<div id="app">
  <router-view></router-view>
</div>
```

```
const User = {
  template: '<div>User {{ $route.params.id }}</div>'
}

const router = new VueRouter({
  routes: [
    { path: '/user/:id', component: User }
  ]
})
```

Ở ví dụ trên, ```<router-view>``` ở cấp cao nhất, sẽ render component được route có cấp cao nhất ánh xa tới. Một components cũng có thể chưa ```<router-view>```. Ví dụ:

```
const User = {
  template: `
    <div class="user">
      <h2>User {{ $route.params.id }}</h2>
      <router-view></router-view>
    </div>
  `
}
```

Để  render một component ở trong component, sử dụng ```children``` option ở trong VueRouter:

```
const router = new VueRouter({
  routes: [
    { path: '/user/:id', component: User,
      children: [
        {
          // UserProfile will be rendered inside User's <router-view>
          // when /user/:id/profile is matched
          path: 'profile',
          component: UserProfile
        },
        {
          // UserPosts will be rendered inside User's <router-view>
          // when /user/:id/posts is matched
          path: 'posts',
          component: UserPosts
        }
      ]
    }
  ]
})
```

Ví dụ cụ thể:

* file HTML:

```
<script src="https://unpkg.com/vue/dist/vue.js"></script>
<script src="https://unpkg.com/vue-router/dist/vue-router.js"></script>

<div id="app">
  <p>
    <router-link to="/user/foo">/user/foo</router-link>
    <router-link to="/user/foo/profile">/user/foo/profile</router-link>
    <router-link to="/user/foo/posts">/user/foo/posts</router-link>
  </p>
  <router-view></router-view>
</div>
```

* file JS:

```
const User = {
  template: `
    <div class="user">
      <h2>User {{ $route.params.id }}</h2>
      <router-view></router-view>
    </div>
  `
}

const UserHome = { template: '<div>Home</div>' }
const UserProfile = { template: '<div>Profile</div>' }
const UserPosts = { template: '<div>Posts</div>' }

const router = new VueRouter({
  routes: [
    { path: '/user/:id', component: User,
      children: [
        // UserHome will be rendered inside User's <router-view>
        // when /user/:id is matched
        { path: '', component: UserHome },
				
        // UserProfile will be rendered inside User's <router-view>
        // when /user/:id/profile is matched
        { path: 'profile', component: UserProfile },

        // UserPosts will be rendered inside User's <router-view>
        // when /user/:id/posts is matched
        { path: 'posts', component: UserPosts }
      ]
    }
  ]
})

const app = new Vue({
	el: "#app",
  router 
})
```

# Named Routes

Trong một vài trường hợp, đặt tên cho ```route``` sẽ thuận tiện hơn. Đặt tên cho ```route``` như sau:
 
 ```
const router = new VueRouter({
  routes: [
    {
      path: '/user/:userId',
      name: 'user',
      component: User
    }
  ]
})
 ```

Truyền ```named route``` vào ```<router-link>```:

```
<router-link :to="{ name: 'user', params: { userId: 123 }}">User</router-link>
```

# Named Views

Đôi khi cần hiển thị nhiều view cũng một thời điểm thay vì hiển thị lồng nhau. Giải pháp tốt nhất là sử dụng ```named views```. ```router-view``` nếu không có ```name``` mặc định sẽ có tên là ```default```. 

```
<router-view class="view one"></router-view>
<router-view class="view two" name="a"></router-view>
<router-view class="view three" name="b"></router-view>
```

Mỗi view sẽ tương ứng với một ```components``` cho cùng một route.

```
const router = new VueRouter({
  routes: [
    {
      path: '/',
      components: {
        default: Foo,
        a: Bar,
        b: Baz
      }
    }
  ]
})
```
