<template>
	<section>
		<!--工具条-->
		<el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
			<el-form :inline="true" :model="filters">
				<el-form-item>
					<el-input v-model="filters.keyword" placeholder="关键字查询"></el-input>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" v-on:click="getProducts">查询</el-button>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" @click="handleAdd">新增</el-button>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" @click="handleViewsPropeties">显示属性</el-button>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" @click="handleSkuProperties">SKU属性</el-button>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" @click="handleOnSale">上架</el-button>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" @click="handleOffSale">下架</el-button>
				</el-form-item>
			</el-form>
		</el-col>

		<!--列表-->
		<el-table :data="products" highlight-current-row v-loading="listLoading" @selection-change="selsChange" style="width: 100%;">
			<el-table-column type="selection" width="55">
			</el-table-column>
			<el-table-column type="index" width="60">
			</el-table-column>
			<el-table-column prop="name" label="标题" min-width="280" sortable>
			</el-table-column>
			<el-table-column prop="subName" label="副标题" min-width="200"  sortable>
			</el-table-column>
			<el-table-column prop="productType.name" label="商品类型" width="120" sortable>
			</el-table-column>
			<el-table-column prop="brand.name" label="品牌" width="150" sortable>
			</el-table-column>
			<el-table-column prop="state" label="状态" min-width="80" sortable>
				<template scope="scope">
					<span style="color: blue;" v-if="scope.row.state==1">上架</span>
					<span style="color: crimson" v-else>下架</span>
				</template>
			</el-table-column>
			<el-table-column prop="onSaleTime" label="上架时间" width="180" sortable :formatter="onSale">
			</el-table-column>
			<el-table-column prop="offSaleTime" label="下架时间" width="180" sortable :formatter="offSale">
			</el-table-column>
			<el-table-column label="操作" width="150">
				<template scope="scope">
					<el-button size="small" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
					<el-button type="danger" size="small" @click="handleDel(scope.$index, scope.row)">删除</el-button>
				</template>
			</el-table-column>
		</el-table>

		<!--工具条-->
		<el-col :span="24" class="toolbar">
			<el-button type="danger" @click="batchRemove" :disabled="this.sels.length===0">批量删除</el-button>
			<el-pagination layout="prev, pager, next" @current-change="handleCurrentChange" :page-size="pageSize" :total="total" style="float:right;">
			</el-pagination>
		</el-col>

		<!--编辑界面-->
		<el-dialog title="编辑" v-model="editFormVisible" :close-on-click-modal="false">
			<el-form :model="editForm" label-width="80px" :rules="editFormRules" ref="editForm">
				<el-form-item label="标题" prop="name">
					<el-input v-model="editForm.name" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="副标题" prop="subName">
					<el-input v-model="editForm.subName" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="商品类型">
					<el-cascader
							v-model="editForm.editSelectedType"
							:change-on-select="true"
							:options="producttypes"
							:show-all-levels="false"
							:props="defaultParams"></el-cascader>
				</el-form-item>
				<el-form-item label="品牌名称">
					<el-select v-model="editForm.brandId" filterable placeholder="请选择品牌">
						<el-option
								v-for="item in brands"
								:label="item.name"
								:value="item.id">
						</el-option>
					</el-select>
				</el-form-item>
				<el-form-item label="媒体属性" prop="medias">
					<el-upload
							class="upload-demo"
							action="http://localhost:9527/services/common/file/"
							:on-remove="handleRemove"
							:on-success="handleupload"
							:before-upload="handleBeforeUpload"
							:file-list="fileList"
							list-type="picture">
						<el-button size="small" type="primary">点击上传</el-button>
					</el-upload>
				</el-form-item>
				<el-form-item label="商品描述">
					<el-input type="textarea" v-model="editForm.ext.description"></el-input>
				</el-form-item>
				<el-form-item label="商品详情">
					<quill-editor
							v-model="editForm.ext.richContent"
							ref="myQuillEditor">
					</quill-editor>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click.native="editFormVisible = false">取消</el-button>
				<el-button type="primary" @click.native="editSubmit" :loading="editLoading">提交</el-button>
			</div>
		</el-dialog>
		<!--SKU属性维护-->
		<el-dialog title="SKU属性"  v-model="skuPropertieFormVisible">
			<el-card class="box-card" v-for="skuProperty in skuProperties">
				<div slot="header" class="clearfix" >
					<span style="line-height: 36px;">{{skuProperty.specName}}</span>
				</div>
				<div v-for="index in skuProperty.options.length+1" class="text item">
					<el-row>
						<el-col :span="18">
							<el-input v-model="skuProperty.options[index-1]" auto-complete="off"></el-input>
						</el-col>
						<el-col :span="6">
							<el-button @click="removeProperty(i,index-1)">删除</el-button>
						</el-col>
					</el-row>
				</div>
			</el-card>
			<div>
				<el-table :data="skus" highlight-current-row style="width: 100%;">
					<el-table-column v-if="key!='price'&&key!='stock'&&key!='indexs'" v-for="(value,key) in skus[0]" :label="key" :prop="key"  width="100%">
					</el-table-column>
					<el-table-column v-if="(key=='price'||key=='stock')&&key!='indexs'" v-for="(value,key) in skus[0]" :label="key" :prop="key"  width="100%">
						<template scope="scope">
							<el-input v-model="scope.row[key]" auto-complete="false"/>
						</template>
					</el-table-column>
				</el-table>
			</div>
			<div slot="footer" class="dialog-footer">
				<el-button @click.native="skuPropertieFormVisible = false">取消</el-button>
				<el-button type="primary" @click.native="updateSkuProperties">提交</el-button>
			</div>
		</el-dialog>
		<!--显示属性维护-->
		<el-dialog title="显示属性"  v-model="viewsPropertieFormVisible">
			<el-form v-for="viewPropertie in viewsProperties" label-width="80px" >
				<el-form-item :label="viewPropertie.specName" >
					<el-input v-model="viewPropertie.value" auto-complete="off"></el-input>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click.native="viewsPropertieFormVisible = false">取消</el-button>
				<el-button type="primary" @click.native="updateViewsPropertie">提交</el-button>
			</div>
		</el-dialog>
		<!--新增界面-->
		<el-dialog title="新增" v-model="addFormVisible" :close-on-click-modal="false">
			<el-form :model="addForm" label-width="80px" :rules="addFormRules" ref="addForm">
				<el-form-item label="标题" prop="name">
					<el-input v-model="addForm.name" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="副标题" prop="subName">
					<el-input v-model="addForm.subName" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="商品类型">
					<el-cascader
							v-model="addForm.selectedType"
							:change-on-select="true"
							:options="producttypes"
							:show-all-levels="false"
							:props="defaultParams"></el-cascader>
				</el-form-item>
				<el-form-item label="品牌名称">
					<el-select v-model="addForm.brandId" filterable placeholder="请选择品牌">
						<el-option
								v-for="item in brands"
								:label="item.name"
								:value="item.id">
						</el-option>
					</el-select>
				</el-form-item>
				<el-form-item label="媒体属性" prop="medias">
					<el-upload
							class="upload-demo"
							action="http://localhost:9527/services/common/file/"
							:on-remove="handleRemove"
							:on-success="handleupload"
							:before-upload="handleBeforeUpload"
							:file-list="fileList"
							list-type="picture">
						<el-button size="small" type="primary">点击上传</el-button>
					</el-upload>
				</el-form-item>
				<el-form-item label="商品描述">
					<el-input type="textarea" v-model="addForm.ext.description"></el-input>
				</el-form-item>
				<el-form-item label="商品详情">
					<quill-editor
							v-model="addForm.ext.richContent"
							ref="myQuillEditor">
					</quill-editor>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click.native="addFormVisible = false">取消</el-button>
				<el-button type="primary" @click.native="addSubmit" :loading="addLoading">提交</el-button>
			</div>
		</el-dialog>
	</section>
</template>

<script>
	import util from '../../common/js/util'
	//import NProgress from 'nprogress'
	import { getUserListPage, removeUser, batchRemoveUser, editUser, addUser } from '../../api/api';

	export default {
		data() {
			return {
				skuPropertieFormVisible:false,
				viewsPropertieFormVisible:false,
				//SKU属性组合table
				skus:[],
				//SKU属性
				skuProperties:[],
				//显示属性
				viewsProperties:[],
				fileList:[],
				selectedType:[],
				editSelectedType:[],
				//商品类型
				producttypes:[],
				defaultParams: {
					label: 'name',
					value: 'id',
					children: 'children'
				},
				//品牌列表
				brands:[],
				filters: {
					keyword: ''
				},
				products: [],
				pageNum: 1,
				pageSize:10,
				total: 0,
				listLoading: false,
				sels: [],//列表选中列

				editFormVisible: false,//编辑界面是否显示
				editLoading: false,
				editFormRules: {
					name: [
						{ required: true, message: '请输入姓名', trigger: 'blur' }
					]
				},
				//编辑界面数据
				editForm: {
					name: '',
					subName: '',
					brandId:'',
					medias:'',
					productTypeId:'',
					selectedType:[],
					ext:{
						description: '',
						richContent: ''
					}
				},

				addFormVisible: false,//新增界面是否显示
				addLoading: false,
				addFormRules: {
					name: [
						{ required: true, message: '请输入姓名', trigger: 'blur' }
					]
				},
				//新增界面数据
				addForm: {
					name: '',
					subName: '',
					brandId:'',
					medias:'',
					productTypeId:'',
					selectedType:[],
					ext:{
						description: '',
						richContent: ''
					}
				}
			}
		},
		methods: {
			//删除SKU属性选项
			removeProperty(){

			},
			//保存Sku属性
			updateSkuProperties(){
				let productId = this.sels[0].id;
				let param={};
				param.skuProperties=this.skuProperties;
				param.skus=this.skus;
				console.debug("param",param);
				this.$http.post("/product/product/saveSkuProperties?productId="+productId,param)
						.then(res=>{
							let{success,message}=res.data;
							if(success){
								this.$message({
									message: '更新成功',
									type: 'success'
								});
								this.skuPropertieFormVisible = false;
							}else {
								this.$message({
									message: message,
									type: 'error'
								});
							}
						})
			},
			//显示属性更新
			updateViewsPropertie(){
				let productId = this.sels[0].id;
				this.$http.post("/product/product/updateViewsProperties?productId="+productId,this.viewsProperties)
						.then(res=>{
							let{success,message}=res.data;
							if(success){
								this.$message({
									message: '更新成功',
									type: 'success'
								});
								this.viewsPropertieFormVisible = false;
							}else {
								this.$message({
									message: message,
									type: 'error'
								});
							}
						})
			},
			//显示属性
			handleViewsPropeties(){
				//打开显示属性模态窗
				let length = this.sels.length;
				if(length===0){
					this.$message({
						message: '请选中一行数据',
						type: 'warning'
					});
					return ;
				}
				if (length>1){
					this.$message({
						message: '只能选中一行数据',
						type: 'warning'
					});
					return ;
				}
				let productId = this.sels[0].id;
				this.viewsPropertieFormVisible=true;
				this.viewsProperties=[];
				this.$http.get("/product/product/getViewsProperties/"+productId)
						.then(res=>{
							this.viewsProperties=res.data;
							console.debug(res.data)
						})

			},
			//SKU属性
			handleSkuProperties(){
				//打开SKU属性模态窗
				let length = this.sels.length;
				if(length===0){
					this.$message({
						message: '请选中一行数据',
						type: 'warning'
					});
					return ;
				}
				if (length>1){
					this.$message({
						message: '只能选中一行数据',
						type: 'warning'
					});
					return ;
				}
				let productId = this.sels[0].id;
				this.skuPropertieFormVisible=true;
				this.skuProperties=[];
				this.$http.get("/product/product/getSkuProperties/"+productId)
						.then(res=>{
							this.skuProperties=res.data;
						})
			},
			//上架
			handleOnSale(){},
			//下架
			handleOffSale(){},

			changeDetSelect(key,treeData){
				let arr = []; // 在递归时操作的数组
				let returnArr = []; // 存放结果的数组
				let depth = 0; // 定义全局层级
				// 定义递归函数
				function childrenEach(childrenData, depthN) {
					for (let j = 0; j < childrenData.length; j++) {
						depth = depthN; // 将执行的层级赋值 到 全局层级
						arr[depthN] = (childrenData[j].id);
						if (childrenData[j].id === key) {
							returnArr = arr.slice(0, depthN+1); //将目前匹配的数组，截断并保存到结果数组，
							break
						} else {
							if (childrenData[j].children) {
								depth ++;
								childrenEach(childrenData[j].children, depth);
							}
						}
					}
					return returnArr;
				}
				return childrenEach(treeData, depth);
			},
			//删除后
			handleRemove(file, fileList){
				console.debug(file);
				let{success,message,resultObject}=file.response;
				console.debug(resultObject);
				this.$http.delete("/common/file?fileid="+resultObject)
						.then(res=>{
							console.debug(res);
							this.fileList=fileList;
						})
			},
			//上传成功后
			handleupload(response, file, fileList){
				console.debug(fileList);
				let{success,message,resultObject}=response;
				if(success){
					this.$message({
						message: '上传成功',
						type: 'success'
					});
				}else {
					this.$message({
						message: message,
						type: 'error'
					});
				}
				this.fileList=fileList;
			},
			//上传之前,判断上传的是否是图片和上传图片的数量
			handleBeforeUpload(file){
				let{type}=file;
				let fileType=type.split("/")[0];
				if(fileType!="image"){
					this.$message({
						message: "只能上传图片",
						type: 'error'
					});
					return false;
				}else {}
			},
			//加载品牌类型
			loadBrands(){
				this.$http.get("/product/brand/list")
						.then(res=>{
							this.brands=res.data;
						})
			},
			//下拉框加载商品类型
			loadTypeTree(){
				this.$http.get("/product/productType/loadTypeTree")
						.then(res=>{
							this.producttypes=this.getTreeData(res.data);
						})
			},
			getTreeData(data){
				// 循环遍历json数据
				for(var i=0;i<data.length;i++){
					if(data[i].children.length<1){
						// children若为空数组，则将children设为undefined
						data[i].children=undefined;
					}else {
						// children若不为空数组，则继续 递归调用 本方法
						this.getTreeData(data[i].children);
					}
				}
				return data;
			},
			//上架时间格式化
			onSale(row, column){
				return this.formateTime(row.onSaleTime);
			},
			//下架时间
			offSale(row, column){
				return this.formateTime(row.offSaleTime);
			},
			//时间格式化
			formateTime(time){
				if (!time){
					return null;
				}
				let date = new Date(time);
				let year = date.getFullYear();
				let month = date.getMonth();
				let day = date.getDate();
				let hour = date.getHours();
				let minute = date.getMinutes();
				let second = date.getSeconds();
				return year+"-"+(month >10 ?month:"0"+month)+"-"+
						(day >10 ? day:"0"+day)+" "+
						(hour>10 ? hour:"0"+hour)+":"+
						(minute>10 ? minute:"0"+minute)+":"+
						(second>10 ? second:"0"+second)
			},
			handleCurrentChange(val) {
				this.page = val;
				this.getProducts();
			},
			//获取商品列表
			getProducts() {
				let para = {
					pageNum: this.page,
					pageSize:this.pageSize,
					keyword: this.filters.keyword
				};
				this.listLoading = true;
				//NProgress.start();
				this.$http.post("/product/product/json",para)
						.then(res=>{
							this.listLoading = false;
							let{total,rows} =res.data;
							this.products=rows;
							console.debug(total);
							this.total=total;
						})
			},
			//删除fastDFS服务器上的图片
			deleteImage(path){
				this.$http.delete("/common/file?fileid="+path)
						.then(console.debug("删掉服务器图片了")).catch();
			},
			//删除
			handleDel: function (index, row) {
				this.$confirm('确认删除该记录吗?', '提示', {
					type: 'warning'
				}).then(() => {
					this.listLoading = true;
					console.debug(row);
					let split =row.medias!=null ? row.medias.split(","):null;
					console.debug(split);
					this.$http.delete("/product/product/delete/"+row.id)
							.then(res=>{
								this.listLoading = false;
								let {success,message}=res.data;
								if (success){
									if (split!=null){
										split.forEach(s=>{
											this.deleteImage(s);
										})
									}
								}else {
									this.$message({
										message: message,
										type: 'error'
									});
								}
							})
				})
			},
			//显示编辑界面
			handleEdit: function (index, row) {
				this.editFormVisible = true;
				this.fileList=[];
				this.editForm = Object.assign({}, row);
				this.editForm.ext={
					description: '',
					richContent: ''
				};
				this.editForm.productType=row.productType;
				this.editForm.editSelectedType=this.changeDetSelect(row.productType.id,this.producttypes);
				this.$http.get("/product/productExt/"+row.id).then(res=>{
					this.editForm.ext=res.data;
				})
			},
			//显示新增界面
			handleAdd: function () {
				this.addFormVisible = true;
				this.fileList=[];
				this.addForm = {
					name: '',
					subName: '',
					brandId:'',
					medias:'',
					productTypeId:'',
					productType: {
						name:'',
						id:''
					},
					selectedType1:[],
					ext:{
						description: '',
						richContent: ''
					}
				};
			},
			//编辑
			editSubmit: function () {
				this.$refs.editForm.validate((valid) => {
					if (valid) {
						this.$confirm('确认提交吗？', '提示', {}).then(() => {
							this.editLoading = true;
							//NProgress.start();
							let para = Object.assign({}, this.editForm);
							console.debug(para);
							this.$http.post("/product/product/add", para)
								.then(res=>{
									this.editLoading = false;
									let {success,message}=res.data;
									if (success){
										this.$message({
											message: '提交成功',
											type: 'success'
										});
										this.$refs['editForm'].resetFields();
										this.editFormVisible = false;
										this.getProducts();
									}else {
										this.$message({
											message: message,
											type: 'error'
										});
									}
								})
						});
					}
				});
			},
			//新增
			addSubmit: function () {
				this.$refs.addForm.validate((valid) => {
					if (valid) {
						this.$confirm('确认提交吗？', '提示', {}).then(() => {
							this.addLoading = true;
							//NProgress.start();
							let para = Object.assign({}, this.addForm);
							para.medias = this.fileList.map(file => (file.response.resultObject)).join(",");
							para.productTypeId=this.addForm.selectedType[this.addForm.selectedType.length-1];
							this.$http.post("/product/product/add", para)
									.then(res => {
										this.addLoading = false;
										let {success,message}=res.data;
										if (success){
											this.$message({
													message: '提交成功',
													type: 'success'
											});
											this.$refs['addForm'].resetFields();
											this.addFormVisible = false;
											this.getProducts();
										}else {
											this.$message({
												message: message,
												type: 'error'
											});
										}
									})
						})
					}
				});
			},
			//选中的行
			selsChange: function (sels) {
				this.sels = sels;
			},
			//批量删除
			batchRemove: function () {
				var ids = this.sels.map(item => item.id).toString();
				this.$confirm('确认删除选中记录吗？', '提示', {
					type: 'warning'
				}).then(() => {
					this.listLoading = true;
					//NProgress.start();
					let para = { ids: ids };
					batchRemoveUser(para).then((res) => {
						this.listLoading = false;
						//NProgress.done();
						this.$message({
							message: '删除成功',
							type: 'success'
						});
						this.getProducts();
					});
				}).catch(() => {

				});
			}
		},
		mounted() {
			this.getProducts();
			this.loadTypeTree();
			this.loadBrands();
		},
		watch:{
			skuProperties:{
				  handler(val,oldVal){
				    //过滤数组张的空数据
					let filtersSku = this.skuProperties.filter(s=>s.options.length>0);
					let result=filtersSku.reduce((pre,cur,currentIndex)=>{
						let tmp=[];
						pre.forEach(p=>{
							cur.options.forEach((o,index)=>{
								let sku=Object.assign({},p);
								sku[cur.specName]=o;
								let lastIndex=sku.indexs;
								if(!lastIndex) lastIndex="";
								if(currentIndex===filtersSku.length-1){
									sku.price=0;
									sku.stock=0;
									sku.indexs=lastIndex+index;
								}else {
									sku.indexs=lastIndex+index+"_";
								}
								sku.indexs=lastIndex;
								tmp.push(sku);
							})
						});
						return tmp;
					},[{}]);
					  this.skus=result;
					  console.debug("result",result);
				  },
				  // 需要深度监听
				  deep: true
				}
		}
	}

</script>

<style scoped>

</style>