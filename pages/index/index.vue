<template>
	<view>
		<form >
			<view class="uni-list">
				<view class="uni-list-cell" style="padding:10px 15px;">
					<wslImg @funcimage="funcimage" @funcavatar="funcavatar" :files_prefix="files_prefix"></wslImg>
				</view>
			</view>
		</form>
	</view>
</template>

<script>
	Date.prototype.Format = function(formatStr)   {
		var str  = formatStr;   
		var Week = ['日','一','二','三','四','五','六'];  
		str=str.replace(/yyyy|YYYY/,this.getFullYear());   
		str=str.replace(/yy|YY/,(this.getYear() % 100)>9?(this.getYear() % 100).toString():'0' + (this.getYear() % 100));   
		str=str.replace(/MM/,this.getMonth()>8?(this.getMonth()+1).toString():'0' + (this.getMonth()+1)); 
		str=str.replace(/M/g,this.getMonth()+1);
		str=str.replace(/w|W/g,Week[this.getDay()]);   
		str=str.replace(/dd|DD/,this.getDate()>9?this.getDate().toString():'0' + this.getDate());   
		str=str.replace(/d|D/g,this.getDate());
		str=str.replace(/hh|HH/,this.getHours()>9?this.getHours().toString():'0' + this.getHours());   
		str=str.replace(/h|H/g,this.getHours());   
		str=str.replace(/mm/,this.getMinutes()>9?this.getMinutes().toString():'0' + this.getMinutes());   
		str=str.replace(/m/g,this.getMinutes());   
		str=str.replace(/ss|SS/,this.getSeconds()>9?this.getSeconds().toString():'0' + this.getSeconds());   
		str=str.replace(/s|S/g,this.getSeconds());   
		return str;   
	}
	import wslImg from "@/components/wsl-img/wsl-img.vue" 
	export default {
		components: {
			wslImg
		},
		data() {
			return {
				files_prefix:'',
				image_src:[]
			}
		},
		onLoad() {
			let time_str1= (new Date()).Format('YYYY-MM-DD');
			this.files_prefix = 'images/' + time_str1+'/';
		},
		methods: {
			funcimage:function(args){
				if( args.type == 'del' ){
					this.image_src.splice(args.index,1);
				}
				if( args.type == 'update' ){
					this.image_src[args.index] = args.data;
				}
				if( args.type == 'add' ){
					this.image_src.push(args.data);
				}
				if( args.type == 'save' ){
					this.image_src = args.data;
				}
			},
			funcavatar:function(data){
				console.log(data);
			}
		}
	}
</script>

<style>
	.content {
		text-align: center;
		height: 400upx;
	}
    .logo{
        height: 200upx;
        width: 200upx;
        margin-top: 200upx;
    }
	.title {
		font-size: 36upx;
		color: #8f8f94;
	}
</style>
