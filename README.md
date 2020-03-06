1. What is VueX?
	- Vuex là thư viện giúp quản lý trạng thái các component trong Vue.js, nó là nơi lưu trữ tập trung cho tất cả các component trong một ứng dụng, với nguyên tắc trạng thái chỉ có thể được thay đổi theo kiểu có thể dự đoán.
2. What is a "State Management Pattern"?
	- Một Vue App cơ bản thường có 3 thành phần chính:
		- State: Trạng thái của ứng dụng.
		- View: Khung nhìn của ứng dụng.
		- Action: Cách thay đổi trạng thái của ứng dụng phản ứng lại khi người dùng thao tác trên view.
	VueX được dùng để quản lí các trạng thái của ứng dụng khi ứng dụng có nhiều component cần sử dụng chung state. Sử VueX sẽ dễ dàng hơn trong việc quản lí state của ứng dụng, dễ dàng maintain và scale-up.
3. State?
	- "Single state tree": Là một object dùng để lưu trữ tất các trạng thái của một ứng dụng. VueX sử dụng "Single state tree" để quản lí tất cả các trạng thái của ứng dụng.
	- Lấy trạng thái của ứng dụng trong Vue Components thông qua thuộc tính "computed":
		```
		computed: {
			count () {
				return store.state.count
			}
		}
		```
	- VueX cung cấp một cơ chế để inject store (store là nơi lưu trữ tất cả  các state) vào trong Vue Component:
		```
		const app = new Vue({
			el: '#app',
			// provide the store using the "store" option.
			// this will inject the store instance to all child components.
			store,
			components: { Counter },
			template: `
			<div class="app">
				<counter></counter>
			</div>
			`
		})
		```
	- Khi sử dụng cơ chế này, cách lấy state của App:
		```
		computed: {
			count () {
				return this.$store.state.count
			}
		}
		```
	- mapState helper: Sử dụng hàm này để lấy một lúc nhiều state của ứng dụng. Tham số truyền vào là tên state được lưu trong store.
		```
		computed: mapState([
			'count'
		])
		```
4. Getters?
	- "getters" trong "store" có tác dụng giống như "computed" trong Vue Components.
	- "getters", tham số truyền vào là "state":
		```
		const store = new Vuex.Store({
			state: {
				todos: [
					{ id: 1, text: '...', done: true },
					{ id: 2, text: '...', done: false }
				]
			},
			getters: {
				doneTodos: state => {
					return state.todos.filter(todo => todo.done)
				}
			}
		})
		```
	- "getters" sẽ nhận các "getters" khác làm thám số thứ 2:
		```
		getters: {
		  // ...
		  doneTodosCount: (state, getters) => {
			return getters.doneTodos.length
		  }
		}
		```
	- Cách gọi getters trong store trong Vue Component:
		```
		computed: {
		  doneTodosCount () {
			return this.$store.getters.doneTodosCount
		  }
		}
		```
	- Cách truyền tham số tới getters bằng cách trả về một function. Đây là một cách hữu hiệu khi thao tác với một mảng trong store:
		```
		getters: {
		  // ...
			getTodoById: (state) => (id) => {
				return state.todos.find(todo => todo.id === id)
		  }
		}
		```
	- "getters" được truy cập thông qua các phương thức sẽ chạy mỗi khi được gọi và kết quả sẽ không được lưu trong bộ nhớ cache.
	- mapGetters helper: Các "getters" trong store sẽ được ánh xạ tới "computed" trong Vue Components.
		```
		import { mapGetters } from 'vuex'

		export default {
		  // ...
		  computed: {
			// mix the getters into computed with object spread operator
			...mapGetters([
			  'doneTodosCount',
			  'anotherGetter',
			  // ...
			])
		  }
		}
		```
5. Mutation?
	- Cách duy nhất để thay đổi "state" trong VueX store là thực hiện "commit" một "mutation". "mutation" trong VueX tương tự như một "events", mỗi "mutation" có kiểu dữ liệu là "string" và một "handler" (hàm xử lí). Trong hàm handler, thực hiện thay đổi state, và tham số truyền vào là "state".
		```
		const store = new Vuex.Store({
		  state: {
			count: 1
		  },
		  mutations: {
			increment (state) {
			  // mutate state
			  state.count++
			}
		  }
		})
		```
	- Không thể gọi trực tiếp "mutation handler", gọi đến "handler" của một "mutaion" sử dụng "store.commit()".
		```
		store.commit('increment');
		```
	- Có thể truyền thêm tham số vào trong "store.commit", hay còn được gọi là "payload":
		```
		mutations: {
		  increment (state, n) {
			state.count += n
		  }
		}

		store.commit('increment', {
		  amount: 10
		})
		```
	- Có một cách khác để thực hiện "commit" một "mutaion" là cách sử dụng trực tiếp một "object" có thuốc tính "type". Khi sử dụng cách này, "handler" của "mutation" vẫn giữ nguyên không thay đổi.
		```
		store.commit({
		  type: 'increment',
		  amount: 10
		})
		```
	- Sử dụng Constatant cho Mutations: Tận sử dụng tối đa một số công cụ, như "linters". Khi đặt hết các hằng số vào một file, dev sẽ có cái nhìn tổng quan về những "mutaion" được sử dụng trong app. Hữu ích khi sử dụng trong project lớn với nhiều dev.
		```
		// mutation-types.js
		export const SOME_MUTATION = 'SOME_MUTATION'
	
		import Vuex from 'vuex'
		import { SOME_MUTATION } from './mutation-types'

		const store = new Vuex.Store({
		state: { ... },
		mutations: {
			// we can use the ES2015 computed property name feature
			// to use a constant as the function name
			[SOME_MUTATION] (state) {
			// mutate state
			}
		}
		})
		```
	- Handler trong "mutations" hoạt động theo cơ chế đồng bộ.
	- Commit Mutations trong Vue Components: Commit mutations trong components bằng this.$store.commit('xxx'), hoặc sử dụng helper mapMutations.
		```
		import { mapMutations } from 'vuex'

		export default {
			methods: {
				...mapMutations([
				'increment', // map `this.increment()` to `this.$store.commit('increment')`

				// `mapMutations` also supports payloads:
				'incrementBy' // map `this.incrementBy(amount)` to `this.$store.commit('incrementBy', amount)`
				]),
				...mapMutations({
				add: 'increment' // map `this.add()` to `this.$store.commit('increment')`
				})
			}
		}
		```
6. Actions?
	- Action tương tự với Mutations, sự khác biệt ở đây:

		- Actions commit mutations.

		- Actions có thể hoạt động theo bất đồng bộ.
	
	```
	const store = new Vuex.Store({
		state: {
			count: 0
		},
		mutations: {
			increment (state) {
			state.count++
			}
		},
		actions: {
			increment (context) {
			context.commit('increment')
			}
		}
	})
	```
	
	- Các "handler" trong "actions" nhận một object "context". Sử dụng "context.commit" để "commit" một mutation hoăcj truy cập "state" và "getters" bằng "context.state" và "context.getters"

	```
	actions: {
		increment ({ commit }) {
			commit('increment')
		}
	}
	```

	- Dispatching Actions: Actions được thực thi bằng phương thức "store.dispatch".
		```store.dispatch('increment')```
	
	- Sử dụng "dispatch" khi handler actions xử lí bất đồng bộ.

		```
		actions: {
			incrementAsync ({ commit }) {
				setTimeout(() => {
				commit('increment')
				}, 1000)
			}
		}
		```

	- Actions cũng hỗ trợ payload trong method dispatch:
	
		```
		store.dispatch('incrementAsync', {
			amount: 10
		})
		// dispatch with an object
			store.dispatch({
			type: 'incrementAsync',
			amount: 10
		})
		```

	- Dispatching Actions in Components: Có thể dispatch actions in components bằng "this.$store.dispatch('xxx')" hoặc sử dụng helper "mapActions"

		```
		import { mapActions } from 'vuex'

		export default {
		// ...
			methods: {
				...mapActions([
				'increment', // map `this.increment()` to `this.$store.dispatch('increment')`

				// `mapActions` also supports payloads:
				'incrementBy' // map `this.incrementBy(amount)` to `this.$store.dispatch('incrementBy', amount)`
				]),
				...mapActions({
				add: 'increment' // map `this.add()` to `this.$store.dispatch('increment')`
				})
			}
		}
		```

7. Modules?
	- VueX cung cấp "modules" để chia "store". Mỗi module bao gồm "state", "mutations", "actions", "getters" hoặc "nested modules".
		```
		const moduleA = {
			state: { ... },
			mutations: { ... },
			actions: { ... },
			getters: { ... }
		}

		const moduleB = {
			state: { ... },
			mutations: { ... },
			actions: { ... }
		}

		const store = new Vuex.Store({
			modules: {
				a: moduleA,
				b: moduleB
			}
		})

		store.state.a // -> `moduleA`'s state
		store.state.b // -> `moduleB`'s state
		```
	- Ở trọng module, tham số đầu tiên truyền vào "mutations", "getter" sẽ là "module's local state".
		```
		const moduleA = {
			state: { count: 0 },
			mutations: {
				increment (state) {
				// `state` is the local module state
				state.count++
				}
			},

			getters: {
				doubleCount (state) {
				return state.count * 2
				}
			}
		}
		```
	- Trong modules, "context.state" will expose the local state, and root state will be exposed as "context.rootState".
		```
		const moduleA = {
			// ...
			actions: {
				incrementIfOddOnRootSum ({ state, commit, rootState }) {
				if ((state.count + rootState.count) % 2 === 1) {
					commit('increment')
				}
				}
			}
		}

		const moduleA = {
			// ...
			getters: {
				sumWithRootCount (state, getters, rootState) {
				return state.count + rootState.count
				}
			}
		}
		```
	- Namespacing: Thông thường, "getters", "setters" trong modules được đăng ký dưới tên mặc định. Sử dụng namespace để  định danh một cách rõ ràng hơn. Đăng ký namespace, trong mỗi module: "namespaced: true".
