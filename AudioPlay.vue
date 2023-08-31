<template>
	<div class="audio-play-module" ref="playerRef"></div>
</template>

<script lang="ts" setup>
import { MusicList } from '@/api/music';
import APlayer from 'APlayer';
import 'APlayer/dist/APlayer.min.css';
import type {PropType} from '@vue/runtime-core';
import {nextTick, onBeforeUnmount, onMounted, defineProps, reactive, ref, toRefs} from 'vue'
const props = defineProps({
  // 开启吸底模式
  fixed: {
    type: Boolean as PropType<boolean>,
    default: true
  },
  // 开启迷你模式
  mini: {
    type: Boolean as PropType<boolean>,
    default: true
  },
  // 音频自动播放
  autoplay: {
    type: Boolean as PropType<boolean>,
    default: false
  },
  // 主题色
  theme: {
    type: String as PropType<string>,
    default: 'rgba(255,255,255,0.2)'
  },
  // 音频循环播放
  loop: {
    type: String as PropType<'all' | 'one' | 'none'>,
    default: 'all'
  },
  // 音频循环顺序
  order: {
    type: String as PropType<'list' | 'random'>,
    default: 'random'
  },
  // 预加载
  preload: {
    type: String as PropType<'auto' | 'metadata' | 'none'>,
    default: 'none'
  },
  // 默认音量
  volume: {
    type: Number as PropType<number>,
    default: 0.5,
    validator: (value: Number) => {
      return value >= 0 && value <= 1;
    }
  },
  // 互斥，阻止多个播放器同时播放，当前播放器播放时暂停其他播放器
  mutex: {
    type: Boolean as PropType<boolean>,
    default: true
  },
  // 传递歌词方式
  lrcType: {
    type: Number as PropType<number>,
    default: 0
  },
  // 列表是否默认折叠
  listFolded: {
    type: Boolean as PropType<boolean>,
    default: true
  },
  // 列表最大高度
  listMaxHeight: {
    type: String as PropType<string>,
    default: '100px'
  },
  // 存储播放器设置的 localStorage key
  storageName: {
    type: String as PropType<string>,
    default: 'aplayer-setting'
  }
})
const playerRef = ref()
const isPlayerShow = ref(false); //播放器是否显示
let instance = null;
const state = reactive({
	pageInfo: {
		page: 1,
		size: 1000,
		total: 0
	},
	musicData: [],
})

// APlayer歌曲信息
const { pageInfo, musicData } = toRefs(state);

const getMusicList = function() {
	let query = {
		//查询推荐并且按是否顶置排序查询
		params: {},
		limit: state.pageInfo.size, //每页条数
		offset: state.pageInfo.page, //当前页
		sort: {
			prop: 'createdAt',
			order: 'desc'
		}
	};
	state.musicData = [];
	state.pageInfo.total = 0;
	MusicList(query).then(res => {
		state.musicData = [];
		state.pageInfo.total = 0;
		if ('count' in res && 'data' in res && res.count && res.data) {
			try {
				res.data.forEach((el, index) => {
					state.musicData.push(el);
				});
			} catch (e) {
				//TODO handle the exception
			}
			if(state.musicData.length){
				isPlayerShow.value = true;
				setTimeout(()=>{
					instance = new APlayer({
						container: playerRef.value,
						fixed: props.fixed,
						mini: props.mini,
						autoplay: props.autoplay,
						theme: props.theme,
						loop: props.loop,
						order: props.order,
						preload: props.preload,
						volume: props.volume,
						mutex: props.mutex,
						lrcType: props.lrcType,
						listFolded: props.listFolded,
						listMaxHeight: props.listMaxHeight,
						storageName: props.storageName,
						audio: state.musicData
					});
				}, 300)
			}
			state.pageInfo.total = res.count;
		}
	});
};

// 初始化
onMounted(() => {
	getMusicList();
})

// 销毁
onBeforeUnmount(() => {
  instance.destroy()
})
</script>

<style lang="scss">
.audio-play-module {
	.aplayer-volume-bar-wrap {
		.aplayer-volume-bar {
			border: thin solid #aaa !important;

			.aplayer-volume {
				background-color: #fff !important;
			}
		}
	}
}
</style>
