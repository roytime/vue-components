<script>
	/**
	 * 这个之前出现过这样的一个bug，就是会出现无限循环，根本的原因是当初的treePrint函数每一次递归
	 * 的返回值push给了数据属性 treeElement，即使在页面中没有显示treeElement，但是因为更新了treeElement
	 * 就导致页面不断的重新渲染，哪怕是0渲染，就导致了无限的循环。
	 * 所以以后递归的值千万不要赋值给vue的数据属性。
	 */
	export default {
		render(createElement) {
			let self = this
			let input = '' 
			//递归返回所有的元素
			let els = this.treePrint(this.tree,createElement)
			/**
			 * 这种做法相当于v-if 
			 */
			/**
			 * createElement有三个参数，分别是
			 * createElement(
				  // {String | Object | Function}
				  // 一个 HTML 标签名、组件选项对象，或者
				  // resolve 了上述任何一种的一个 async 函数。必填项。
				  'div',

				  // {Object}
				  // 一个与模板中属性对应的数据对象。可选。
				  {
					// (详情见下一节)
				  },

				  // {String | Array}
				  // 子级虚拟节点 (VNodes)，由 `createElement()` 构建而成，
				  // 也可以使用字符串来生成“文本虚拟节点”。可选。
				  [
					'先写一些文字',
					createElement('h1', '一则头条'),
					createElement(MyComponent, {
					  props: {
						someProp: 'foobar'
					  }
					})
				  ]
				)
				这个函数会自动检测就第二个参数，如果第二个参数不是对象，那么就当成是第三个参数
			 */
			if(this.inputShow){
				input = createElement(
					'input',
					{
						//这个value属性，如果是写成attrs的，那么就无法动态更新
						domProps:{
							value:self.value
						},
						on:{
							keyup:self.enter
						}
				})
			}
			/**
			 * createElement的第二个参数，数据对象
			 * {
				  // 与 `v-bind:class` 的 API 相同，
				  // 接受一个字符串、对象或字符串和对象组成的数组
				  'class': {
					foo: true,
					bar: false
				  },
				  // 与 `v-bind:style` 的 API 相同，
				  // 接受一个字符串、对象，或对象组成的数组
				  style: {
					color: 'red',
					fontSize: '14px'
				  },
				  // 普通的 HTML attribute
				  attrs: {
					id: 'foo'
				  },
				  // 组件 prop
				  props: {
					myProp: 'bar'
				  },
				  // DOM 属性
				  domProps: {
					innerHTML: 'baz'
				  },
				  // 事件监听器在 `on` 属性内，
				  // 但不再支持如 `v-on:keyup.enter` 这样的修饰器。
				  // 需要在处理函数中手动检查 keyCode。
				  on: {
					click: this.clickHandler
				  },
				  // 仅用于组件，用于监听原生事件，而不是组件内部使用
				  // `vm.$emit` 触发的事件。
				  nativeOn: {
					click: this.nativeClickHandler
				  },
				  // 自定义指令。注意，你无法对 `binding` 中的 `oldValue`
				  // 赋值，因为 Vue 已经自动为你进行了同步。
				  directives: [
					{
					  name: 'my-custom-directive',
					  value: '2',
					  expression: '1 + 1',
					  arg: 'foo',
					  modifiers: {
						bar: true
					  }
					}
				  ],
				  // 作用域插槽的格式为
				  // { name: props => VNode | Array<VNode> }
				  scopedSlots: {
					default: props => createElement('span', props.text)
				  },
				  // 如果组件是其它组件的子组件，需为插槽指定名称
				  slot: 'name-of-slot',
				  // 其它特殊顶层属性
				  key: 'myKey',
				  ref: 'myRef',
				  // 如果你在渲染函数中给多个元素都应用了相同的 ref 名，
				  // 那么 `$refs.myRef` 会变成一个数组。
				  refInFor: true
				}
			 */
			return createElement('div',{
				class:'big',
				on:{
					dblclick:self.dbClick
				},
				attrs:{
					"data-id":0
				}
			},[
				createElement('div',{on:{dblclick:self.dbClick}},[...els]),
				input
			]
			)
		},
		data(){
			return {
				inputShow:false,
				value:'',
				dbs:[],
				treeElement:[],
				id:7
			}
		},
		computed:{
			/**
			 * 原始的数据为[{id:1,name:'name1',pid:0},{id:2,name:'name2',pid:1}...]
			 * 中间改成这样的{
			 * 	"1(这个是pid)":[{id:2,name:'name2',pid:1},...]
			 * }
			 * 最后变成这样的{
			 *   [{id:1,name:'name1',pid:0,children:[{id:2,name:'name2',pid:1}]}]
			 * }
			 */
			tree(){
				let rootObj = {}
				this.dbs.forEach(item=>{
					rootObj[item.pid] = rootObj[item.pid] || []
					rootObj[item.pid].push(item)
				})
				let treeArr = []
				for(let pid in rootObj){
					rootObj[pid].forEach((item)=>{
						this.checkChild(rootObj,item)
						treeArr.push(item)
					})
				}
				//console.log(treeArr,'this.treeArr')
				return treeArr
			}
		},
		methods:{
			//页面或者元素双击就会出现编辑框
			dbClick(e){
				this.inputShow = true
				this.id = e.target.dataset.id
			},
			/**
			 * 输入框敲击回车的处理函数
			 * @param {Object} e  内部事件对象
			 */
			enter(e){
				//console.log(e.keyCode)
				if(e.keyCode==13){
					let v = e.target.value.trim()
					if(v.length == 0) {
						console.log('need input')
						return ''
					}
					//通过这样的方法来模拟获取一个尽量不会重复的id
					let randomStr = Math.random().toString(36).slice(-10)
					let obj = {id:randomStr,name:v,pid:this.id}
					//console.log(obj)
					this.dbs.push(obj)
					this.value = ''
					this.inputShow = false
				}
			},
			//递归打印
			/**
			 * @param {Object} tree
			 * @param {Object} createElement
			 * 这个里面的tree2函数里面的level，没有采用变量，而是直接传递数值的做法
			 * 这个做法值得注意。如果不传入变量，就不会出现缓存结果，也就不会出错
			 */
			treePrint(tree,createElement){
				let elements = []
				//let level = 1
				tree2(tree,1)
				function tree2(t2,level){
					t2.forEach(item=>{
						//console.log(item.name)
						let str  = 'I'
						for(let i=0,len=level*3;i<len;i++){
							str += '-'
						}
						elements.push(createElement('div',{attrs:{"data-id":item.id}},[str+item.name]))
						if(item.children){
							tree2(item.children,level+1)
						}
					})
				}
				console.log(elements)
				return elements
			},
			//递归树
			checkChild(root,item){
				let idStr = item.id + ''
				if(root[idStr]){
					item['children'] = root[idStr]
					delete root[idStr]
					//console.log('incheckchild',item)
					item['children'].forEach((t)=>{
						this.checkChild(root,t)
					})
				}
				//return parent
			}
		}
	}
</script>

<style>
	.big{
		height: 100vh; width: 100%;
		background-color: papayawhip;
	}
</style>

