<template>
	<view class="content">
		<view :style="{height:scrollH}">
			<image class="beijing" src="/static/beijing.png"></image>
			<view class="kapianview">
				<view class="jsqhfhan">
					<image class="cxksbut" src="/static/chongxinks.png" @click="cxksbut()"></image>
					<text class="kndjxc">{{kndjxc}}</text>
					<text class="fenshut" :style="{'transform': defen ? 'scale(2)' : 'scale(1)'}">{{fenshu}}分</text>
				</view>
				<image class="shangpic" mode="widthFix"
					:class="'pic'+item.id"
					:style="{'opacity':item.location[2] ,'left':item.location[0]+'px' ,'top':item.location[1]+'px' ,'z-index':item.cengji}"
					v-for="(item,index) in kapian"
					:key="index"
					v-if="item.weizhi==1 && !item.del"
					:src="item.pic"
					@click="xuanzeimg(index)"></image>
			</view>
			<view class="daixiaochuview">
				<image class="xiapic" mode="widthFix"
					:style="{'opacity':item.location[2]}"
					:src="item.pic" 
					v-for="(item,index) in kapian" 
					:key="index" 
					v-if="item.weizhi==2 && !item.del"></image>
			</view>
		</view>
		<view>
			<uni-popup ref="kndjpopup" :is-mask-click="false">
				<view class="kndjview">
					<button type="default" plain="true" @click="xzkndengji(30)">简单</button>
					<button class="zdbut" type="primary" plain="true" @click="xzkndengji(90)">中等</button>
					<button type="warn" plain="true" @click="xzkndengji(180)">困难</button>
				</view>
			</uni-popup>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				kapian: [],
				pagewidth: 0,
				pageheight: 0,
				jinzdj: false,
				dsnumsb: 8,
				kndengji: 0,
				liubian: 30,
				picbl: 327 / 233,
				fenshu: 0,
				kndjxc: '选择游戏等级',
				kpnum: 0,
				defen: false,
			}
		},
		onReady() {
			var that = this;
			var sys = uni.getSystemInfoSync();
			that.pagewidth = Number(sys.windowWidth / 2);
			uni.createSelectorQuery().in(that).select('.kapianview').boundingClientRect(data => {
				that.pageheight = Number(data.height);
			}).exec();
			that.$refs.kndjpopup.open('center');
		},
		computed:{
			scrollH(){
				var sys = uni.getSystemInfoSync();
				return sys.windowHeight + 'px';
			},
		},
		methods: {
			xuanzeimg(sy){
				var that = this;
				if(that.jinzdj) return;
				that.jinzdj = true;
				var kparr = that.kapian;
				for(var i=0;i<kparr.length;i++){
					if(kparr[i].weizhi == 1 && i != sy && kparr[i].cengji > kparr[sy].cengji && that.shifouxj(sy ,i)){
						that.jinzdj = false;
						return;
					}
				}
				that.yinxiao("dianji");
				kparr[sy].location = [that.pagewidth ,that.pageheight ,1];
				kparr[sy].cengji = 999;
				setTimeout(() => {
					var sfysc = false;
					kparr[sy].weizhi = 2;
					var xcarr = {};
					for(var i=0;i<kparr.length;i++){
						if(kparr[i].weizhi == 2 && kparr[i].del == false){
							var xcsy = kparr[i].name;
							xcarr[xcsy] = xcarr[xcsy]==undefined ? 1 : xcarr[xcsy] + 1;
						}
					}
					for(var key in xcarr){
						if(xcarr[key] >= 3){
							sfysc = true;
							for(var i=0;i<kparr.length;i++){
								if(kparr[i].weizhi == 2 && kparr[i].name == key){
									kparr[i].location[2] = 0;
									that.fenshu = that.fenshu + 1;
									that.delpic(i);
								}
							}
							that.defen = true;
							that.yinxiao("defen");
							that.dfhuany();
						}
					}
					if(!sfysc){
						that.jinzdj = false;
						var wz2 = 0;
						var wz1 = 0;
						for(var i=0;i<kparr.length;i++){
							if(kparr[i].weizhi == 2 && kparr[i].del == false){
								wz2++;
							}
							if(kparr[i].weizhi == 1 && kparr[i].del == false){
								wz1++;
							}
						}
						if(wz2 == that.dsnumsb){
							that.shibai();
						}else{
							if(wz1 == 0 && wz2 != 0){
								that.xzkndengji(that.kpnum);
							}
						}
					}
				}, 300);
			},
			delpic(xs){
				var that = this;
				setTimeout(() => {
					that.kapian[xs].del = true;
				}, 1000);
				setTimeout(() => {
					var wz2 = 0;
					var wz1 = 0;
					for(var i=0;i<that.kapian.length;i++){
						if(that.kapian[i].weizhi == 2 && that.kapian[i].del == false){
							wz2++;
						}
						if(that.kapian[i].weizhi == 1 && that.kapian[i].del == false){
							wz1++;
						}
					}
					if(wz2 == that.dsnumsb){
						that.shibai();
					}else{
						if(wz1 == 0 && wz2 != 0){
							that.xzkndengji(that.kpnum);
						}
					}
					that.jinzdj = false;
				}, 1300);
			},
			dfhuany(){
				var that = this;
				setTimeout(() => {
					that.defen = false;
				}, 300);
			},
			shibai(){
				this.yinxiao("jieshu");
				uni.showModal({
					title: '提示',
					content: '你TM失败了',
					showCancel: false,
					success: function (res) {
						location.reload();
					}
				});
			},
			shifouxj(sy ,xh){
				var that = this;
				var div1 = '';
				var div2 = '';
				uni.createSelectorQuery().in(this).select('.pic'+that.kapian[sy].id).boundingClientRect(data => {
					div1 = data;
				}).exec();
				uni.createSelectorQuery().in(this).select('.pic'+that.kapian[xh].id).boundingClientRect(data => {
					div2 = data;
				}).exec();
				div1.width = div1.width - 4;
				div1.height = div1.height - 4;
				div1.left = div1.left - 4;
				div1.top = div1.top - 4;
				div2.width = div2.width - 4;
				div2.height = div2.height - 4;
				div2.left = div2.left - 4;
				div2.top = div2.top - 4;
				var cPos1 = new Array(div1.width ,div1.height ,div1.left+div1.width/2 ,div1.top+div1.height/2);
				var cPos2 = new Array(div2.width ,div2.height ,div2.left+div2.width/2 ,div2.top+div2.height/2);
				if(Math.abs(cPos1[2] - cPos2[2]) < (cPos1[0] + cPos2[0]) / 2 && Math.abs(cPos1[3] - cPos2[3]) < (cPos1[1] + cPos2[1]) / 2){
					return true;
				}
				return false;
			},
			random(min, max) {
				return Math.floor(Math.random() * (max - min)) + min;
			},
			xzkndengji(num){
				var that = this;
				if(num == 30){
					that.kndjxc = '简单';
				}else if(num == 90){
					that.kndjxc = '中等';
				}else if(num == 180){
					that.kndjxc = '困难';
				}
				that.kpnum = num;
				that.yinxiao("jihuo");
				that.$refs.kndjpopup.close();
				var sys = uni.getSystemInfoSync();
				var pickd = sys.windowWidth / 8;
				var picgd = that.picbl * pickd;
				var zgdsc = Number(num / 4);
				var jzp = Number(num / 6);
				var jzphd = jzp > 27 ? 27 : jzp;
				for(var i=0;i<num;i++){
					var na = that.random(1, jzphd);
					var cj = that.random(0, zgdsc);
					var x = that.random(that.liubian, Number(sys.windowWidth - pickd - that.liubian));
					var y = that.random(that.liubian + that.liubian, Number(that.pageheight - picgd - that.liubian));
					var itme = {id:'a'+i ,name:'a'+na ,pic:"/static/a"+na+".png" ,cengji:cj ,weizhi:1 ,location:[x ,y ,1] ,del:false};
					that.kapian.push(itme);
				}
			},
			cxksbut(){
				location.reload();
			},
			yinxiao(name){
				var tishi = uni.createInnerAudioContext();
				tishi.src = "/static/" + name + ".mp3";
				tishi.play();
			},
		}
	}
</script>

<style>
	.beijing{
		position: fixed;
		top: 0;
		bottom: 0;
		left: 0;
		right: 0;
		width: 100%;
		height: 100%;
		z-index: -1;
	}
	.kapianview{
		position: relative;
		height: 80%;
	}
	.daixiaochuview{
		position: relative;
		height: 20%;
	}
	.shangpic{
		position: absolute;
		width: 12.5%;
		transition: opacity 300ms ease 0ms, top 300ms ease 0ms, left 300ms ease 0ms;
	}
	.xiapic{
		width: 12.5%;
		transition: opacity 1000ms ease 0ms, top 300ms ease 0ms, left 300ms ease 0ms;
	}
	.kndjview{
		width: 400rpx;
		background-color: #9eff90;
		padding: 50rpx;
		border-radius: 5px;
	}
	.zdbut{
		margin: 50rpx 0;
	}
	.jsqhfhan{
		height: 45px;
		width: 100%;
		position: absolute;
		top: 0;
		left: 0;
		right: 0;
		z-index: 1;
		display: flex;
		flex-direction:row;
		flex-wrap:nowrap;
		justify-content:space-between;
		align-items: center;
	}
	.cxksbut{
		height: 25px;
		width: 25px;
		margin: 10px;
	}
	.fenshut{
		margin: 10px;
		height: 25px;
		font-size: 20px;
		font-weight: bold;
		color: #ff6600;
		transition: transform 300ms ease 0ms;
	}
	.kndjxc{
		font-size: 20px;
		font-weight: bold;
		color: #ff00ff;
	}
</style>
