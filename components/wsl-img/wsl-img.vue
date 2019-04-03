<template name="wsl-img">
	<view v-if="type!='avatar'">
		<view class="uni-uploader">
			<view class="uni-uploader-head">
				<view class="uni-uploader-title">点击预览图片</view>
				<view class="uni-uploader-info">{{upload_images_list.length}}/{{count}}</view>
			</view>
			<view class="uni-uploader-body">
				<view class="uni-uploader__files">
					<block v-for="(image,index) in upload_images_list" :key="index">
						<view v-if = " image.image_src !== '' ">
							<view class="uni-uploader__file" style="position: relative;">
								<image class="uni-uploader__img" :src="image.image_src" @tap="previewImage(index)"></image>
								<view class="close-view" @click="close(index)">x</view>
							</view>
						</view>
					</block>
					<view class="uni-uploader__input-box" v-show="upload_images_list.length < count">
						<view class="uni-uploader__input" @click="chooseImg"></view>
					</view>
				</view>
			</view>
		</view>
	</view>
	<view v-else-if="type=='avatar'">
		<view>
			<image style="width:80px;height:80px;" :src="avatar_image" @click="chooseImg" :mode="avatar_mode"></image>
		</view> 
	</view>
</template>

<script>
	var image_obj = {
		vue_obj:{},
		token:'',
		image_url:'',
		qiniu_config:{},
		setVueObj:(obj)=>{
			image_obj.vue_obj = obj;
		},
		setQiniuConfig:(obj)=>{
			image_obj.qiniu_config = obj;
		},
		create:(_vue_obj) => {
			let _self     = image_obj;
			_self.vue_obj = _vue_obj;
			let count     = _self.vue_obj.count;
			if( _self.vue_obj.type === 1){
				_self.vue_obj.upload_images_list = [];
				_self.vue_obj.images_list = [];
				count = 1;
			}
			uni.chooseImage({
				count: count - _self.vue_obj.upload_images_list.length,
				sizeType: ['compressed'],
				sourceType: ['album', 'camera'],
				success(res) {
					for (let i in res.tempFiles) {
						res.tempFiles[i]['upload_percent'] = 0
						res.tempFiles[i]['image_src']      = ''
						_self.vue_obj.upload_images_list.push( res.tempFiles[i] );
						_self.vue_obj.images_list.push('');
						_self.vue_obj.$emit('funcimage', {
							type:'add',data:'',index:''
						});
					}
					if(_self.vue_obj.type===0){
						if( _self.vue_obj.upload_images_list.length > count ){
							_self.vue_obj.upload_images_list = _self.vue_obj.upload_images_list.slice(0, count);
							_self.vue_obj.images_list        = _self.vue_obj.images_list.slice(0, count);
							_self.vue_obj.$emit('funcimage', {
								type:'save',data:_self.vue_obj.images_list,index:''
							});
						}
					} else {
						_self.vue_obj.$emit('funcimage', {
							type:'save',data:_self.vue_obj.images_list,index:''
						});
					}
					image_obj.upload();
				}
			})
		},
		get_token:(func_callback) => {
			let config = image_obj.qiniu_config;
			uni.request({
				url:config.uplode_token_url, //仅为示例，并非真实接口地址。
				data:config.upload_token_url_data,
				header:config.upload_token_url_header,
				dataType:'json',
				success: (res) => {
					if( res.statusCode === 200 ){
						let data = res.data;
						image_obj.token     = data.uptoken;
						image_obj.image_url = data.url;
						if( typeof func_callback !=='undefined' ){
							func_callback(data);	
						}
					}
				}
			});
		},
		upload:() => {
			let vue = image_obj.vue_obj;
			for (let index in vue.upload_images_list) {
				if (vue.upload_images_list[index]['upload_percent'] == 0) {
					image_obj.send_data(vue.upload_images_list,index);
				}
			}
		},
		getNewFileName:(path,index)=>{
			let path_ext     = path.substring(path.lastIndexOf('.') + 1);
			let files_prefix = image_obj.vue_obj.files_prefix;
			let filename     = image_obj.vue_obj.filename;
			let str          = files_prefix + filename + '_' + index + '.' + path_ext;
			return str;
		},
		send_data:(images_list,index) => {
			let _self    = image_obj;
			let config   = _self.qiniu_config;
			let path     = images_list[index]['path'];
			let key      = _self.getNewFileName(path,index);
			let form     = {
				token:image_obj.token,
				key:key,
			};
			const upload_task = uni.uploadFile({
				url:image_obj.vue_obj.upload_url,
				filePath: images_list[index]['path'],
				name: 'file',
				formData:form,
				success(res) {
					if ( res.statusCode == 200 ) {
						 let data = JSON.parse(res.data);
					     let image_src = _self.image_url + '/' + data.key;
						 _self.vue_obj.upload_images_list[index]['image_src'] = image_src;
						 _self.vue_obj.images_list[index] = image_src;
						 _self.vue_obj.$emit('funcimage',{
							type:'update',data:image_src,index:index
						});
						if( index === images_list.length - 1 ){
							_self.vue_obj.getNewFileName();
							_self.get_token();
						}
						if( _self.vue_obj.type == 1 ){
							_self.vue_obj.upload_images_list = [];
							_self.vue_obj.images_list        = [];
							_self.vue_obj.setAvatarImage(image_src);
						}
					} 
				},
				fail(err) {
					uni.showLoading({title: `上传失败!`})
					setTimeout(() => {uni.hideLoading();}, 2000)
					console.log(JSON.stringify(err));
				}
			});
			upload_task.onProgressUpdate((res) => {
				_self.vue_obj.upload_images_list[index]['upload_percent'] = res.progress;
			})
		}
	};
	
	export default {
		name: 'wsl-img',
		data() {
			return {
				upload_url:'https://upload.qiniup.com',
			};
		},
		props: {
			'type':{
				type:String,
				default:'images'
			},
			'count':{
				type:[Number, String],
				default:9
			},
			'files_prefix':{
				type:String,
				default:'images/'
			},
			'qiniu_config':{
				type:Object,
				default:() => {
					return {
						uplode_token_url:'http://test.api.nijiasheji.com/uploader/getQiniuToken',
						upload_token_url_header:{},
						upload_token_url_data:{}
					}
				}
			},
			'upload_images_list':{
				type:Array,
				default:[]
			},
			'images_list':{
				type:Array,
				default:[]
			},
			'avatar_image':{
				type:String,
				default:'/static/images/01.jpg'
			},
			'avatar_mode':{
				type:String,
				default:'aspectFill'
			},
			'filename':{
				type:String,
				default:''
			},
		},
		methods: {
			chooseImg() { //选择图片,完成上传
				image_obj.create(this);
			},
			previewImage(index) { //预览图片
				let images_list = this.images_list;
			    uni.previewImage({
			        urls: images_list,
					current:images_list[index]
			    });
			},
			close(e){
			    this.upload_images_list.splice(e,1);
				this.images_list.splice(e,1);
				this.$emit('funcimage',{
					type:'del',data:'',index:e
				});
			},
			getFilename(){
				this.filename = (new Date()).getTime();
			},
			setAvatarImage(avatar_image){
				this.avatar_image = avatar_image;
				this.$emit('funcavatar',avatar_image);
			}
		},
		onShow() {
			this.getFilename();
			image_obj.setVueObj(this);
			image_obj.setQiniuConfig(this.qiniu_config);
			image_obj.get_token();
		}
	}
</script>
<style>
	.close-view{
	    text-align: center;
		line-height:14px;
		height: 16px;
		width: 16px;
		border-radius: 50%;
		background: #FF5053;
		color: #FFFFFF;
		position: absolute;
		top: -6px;
		right: -4px;
		font-size: 12px;
	}
</style>
