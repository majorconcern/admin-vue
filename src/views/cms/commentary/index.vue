<template>
	<div>
		<el-row :gutter="10">
			<el-col :span="3">
				<el-select
					style="width: 100%"
					clearable
					v-model="params.video_type"
					placeholder="视频类型"
				>
					<el-option label="全部" value="" />
					<el-option label="个人发布" :value="0" />
					<el-option label="机构发布" :value="1" />
				</el-select>
			</el-col>
			<el-col :span="5">
				<el-input
					v-model.trim="params.phone"
					placeholder="发布者手机号"
					prefix-icon="el-icon-search"
					clearable
				/>
			</el-col>
			<el-col :span="5">
				<el-input
					v-model.trim="params.commentary_title"
					placeholder="解说标题"
					prefix-icon="el-icon-search"
					clearable
				/>
			</el-col>
			<el-col :span="11">
				<el-button
					class="fr"
					style="margin-left: 10px;"
					type="primary"
					@click="$refs.dataTable.handleChange(true)"
					>搜索</el-button
				>
				<el-button class="fr" @click="$refs.dataTable.backendPagingFunc()">刷新</el-button>
			</el-col>
		</el-row>
		<el-row :gutter="10" style="margin-top: 10px; " v-if="params.commentary_type == 2">
			<el-col :span="3">
				<el-select
					style="width: 100%"
					clearable
					v-model="params.online_status"
					placeholder="上线状态"
				>
					<el-option label="全部" value="" />
					<el-option label="上线" :value="0" />
					<el-option label="下线" :value="1" />
				</el-select>
			</el-col>
			<el-col :span="5" v-if="params.commentary_type == 2">
				<el-select
					v-model="movieSelect.movie"
					value-key="movie_id"
					placeholder="输入影片名称 选择影片进行搜索"
					filterable
					clearable
					no-data-text="暂无匹配影片数据"
					no-match-text="暂无匹配影片数据"
					remote
					reserve-keyword
					:remote-method="toGetMovieList"
					v-loadmore:[movieSelect.direction]="loadMoreMovie"
					@clear="
						(movieSelect.movieList = []),
							(movieSelect.page_number = 1),
							(movieSelect.loadAll = false)
					"
					popper-class="index"
					style="width: 100%; "
				>
					<el-option
						v-for="item in movieSelect.movieList"
						:key="item.movie_id"
						:label="item.movie_name"
						:value="item"
					>
						<span style="float: left">{{ item.movie_name }}</span>
						<span style="float: right; color: #8492a6; font-size: 13px">{{
							item.movie_id
						}}</span>
					</el-option>
					<div
						v-if="movieSelect.movieLoading"
						style="width: 100; text-align: center; color: #9f9f9f; font-size: 13px"
					>
						加载中...
					</div>
					<div
						v-if="movieSelect.loadAll"
						style="width: 100; text-align: center; color: #9f9f9f; font-size: 13px"
					>
						加载完毕
					</div>
				</el-select>
			</el-col>
		</el-row>
		<el-row>
			<commentary-add
				v-if="params.commentary_type == 3"
				:user-name="user_name"
				:on-success="$refs.dataTable.backendPagingFunc"
				style="display: block; margin-top: 10px; "
			/>
		</el-row>

		<dynamic-table
			:currentPage="1"
			:pageSizes="[5, 10, 20, 50, 100, 200]"
			:pageSize="10"
			:background="true"
			:paging="true"
			:backendPaging="true"
			@backendPagingFunc="queryTableData"
			:dynamicColumnSetting="true"
			:showAlwaysShowColumnInCheckbox="true"
			:border="true"
			ref="dataTable"
		>
			<el-table-column
				prop="id"
				label="#"
				key="1"
				width="50"
				v-if="params.commentary_type == 0"
			/>
			<el-table-column
				prop="original_id"
				label="#"
				key="2"
				width="50"
				v-if="params.commentary_type != 0"
			/>
			<el-table-column prop="phone" label="发布者手机号" width="120" key="4" />
			<el-table-column
				prop="video_nickname"
				label="原始发布者昵称"
				key="5"
				v-if="params.commentary_type != 0"
			/>
			<el-table-column
				prop="nickname"
				label="原始发布者昵称"
				key="55"
				v-if="params.commentary_type == 0"
			/>

			<el-table-column prop="title" label="标题" key="6" v-if="params.commentary_type != 0" />
			<el-table-column
				prop="description"
				label="描述"
				key="7"
				v-if="params.commentary_type != 0"
				min-width="220"
			/>

			<el-table-column
				prop="video_title"
				label="原始标题"
				key="666"
				v-if="params.commentary_type == 0"
			/>
			<el-table-column
				prop="video_description"
				label="原始描述"
				key="777"
				v-if="params.commentary_type == 0"
				min-width="220"
			/>

			<el-table-column
				prop="movie_id"
				label="影片"
				key="77"
				min-width="90"
				v-if="params.commentary_type != 0"
			>
				<template slot-scope="scope">
					<p v-if="scope.row.movie_id" class="p0 m0">id: {{ scope.row.movie_id }}</p>
					<p v-if="scope.row.movie_name" class="p0 m0">
						片名: 《{{ scope.row.movie_name }}》
					</p>
				</template>
			</el-table-column>
			<el-table-column prop="video_type" label="发布类型" key="8" width="90">
				<template slot-scope="scope">
					<el-tag :type="scope.row.video_type == 1 ? 'warning' : 'primary'">
						{{ scope.row.video_type == 1 ? "机构发布" : "个人发布" }}
					</el-tag>
				</template>
			</el-table-column>
			<el-table-column
				prop="online_status"
				label="上线类型"
				v-if="params.commentary_type == 2"
				key="9"
				width="100"
			>
				<template slot-scope="scope">
					<el-button
						:type="scope.row.online_status == 0 ? 'success' : 'primary'"
						style="padding: 3px 6px;"
						@click="toAudit(scope.row)"
					>
						{{
							scope.row.online_status == 0
								? "上线"
								: scope.row.back_status == 1
								? "下线后打回"
								: "下线"
						}}
					</el-button>
					<p
						v-if="scope.row.online_status == 1"
						style="margin-left: 5px; cursor: pointer;"
						@click="toGetOfflineList(scope.row)"
					>
						查看<i class="el-icon-d-arrow-right"></i>
					</p>
				</template>
			</el-table-column>
			<el-table-column prop="title_page_url" label="封面" key="10" width="110">
				<template slot-scope="scope">
					<el-image
						v-if="
							scope.row.title_page_url != null && scope.row.title_page_url.length > 0
						"
						:src="scope.row.title_page_url"
						@click="previewImgMore(0, [scope.row.title_page_url])"
						fit="contain"
					/>
					<span v-else>
						无图片
					</span>
				</template>
			</el-table-column>
			<el-table-column prop="create_time" label="创建时间" key="11" />
			<el-table-column
				label="操作"
				align="center"
				:width="params.commentary_type == 0 ? 280 : 130"
				key="12"
			>
				<template slot-scope="scope">
					<el-button
						v-if="params.commentary_type == 0"
						type="primary"
						@click="jumpOtherPage('video', scope.row.source_id)"
						>视频结果</el-button
					>
					<el-button
						v-if="params.commentary_type == 0"
						type="primary"
						style="margin: 0 5px 0 5px; "
						@click="jumpOtherPage('audio', scope.row.source_id)"
						>音频结果</el-button
					>
					<commentary-detail
						:data="scope.row"
						:user-name="user_name"
						:commentary-type="params.commentary_type"
						:on-success="$refs.dataTable.backendPagingFunc"
					/>
				</template>
			</el-table-column>
		</dynamic-table>

		<el-dialog title="下线操作记录" :visible.sync="visibleDialog">
			<el-table :data="offlineList" border>
				<el-table-column type="index" label="#" />
				<el-table-column prop="operate_reason" label="原因" />
				<el-table-column prop="operate_user" label="操作人" width="100" />
				<el-table-column prop="create_time" label="创建时间" width="150" />
			</el-table>
		</el-dialog>
	</div>
</template>

<script>
import {
	getMovieCommentaryList,
	updateMovieCommentary,
	getOfflineRecords
} from "@/api/cms/commentary/commentary";
import { getMovieList } from "@/api/cms/commentary/movie";

import CommentaryDetail from "./components/Detail.vue";
import CommentaryAdd from "./components/Add.vue";

export default {
	name: "MovieCommentary",
	components: {
		CommentaryDetail,
		CommentaryAdd
	},
	data() {
		return {
			loading: false,
			total: 0,
			tableData: [],
			params: {
				commentary_title: "",
				online_status: "",
				source_id: "",
				phone: "",
				video_type: "",
				movie_id: "",
				commentary_type: 0
			},
			selectAll: false,
			movieSelect: {
				movie: "",
				movie_id: "",
				movie_name: "",
				page_number: 1,
				page_size: 10,
				movieList: [],
				movieLoading: false,
				direction: "index",
				loadAll: false
			},
			visibleDialog: false,
			offlineList: [],
			user_name: "test"
		};
	},
	methods: {
		previewImgMore(index, imgList) {
			this.$hevueImgPreview({
				multiple: true,
				imgList,
				nowImgIndex: index,
				keyboard: true
			});
		},
		jumpOtherPage(type, source_id) {
			window.open(
				`${process.env.VUE_APP_PAGE}/plan_media/to_media_${type}_result?sourceId=${source_id}`
			);
		},
		queryTableData(page_number, page_size, callback) {
			let commentary_type = this.$route.path.substring(1).split("/")[2] || 0;
			this.$set(this.params, "commentary_type", commentary_type);
			let { movie: { movie_id = "" } = {} } = this.movieSelect;
			getMovieCommentaryList({
				...this.params,
				movie_id,
				page_number,
				page_size
			}).then(res => {
				if (res.code == "000000") {
					callback({
						data: res.content.data,
						data_total: res.content.data_total
					});
				}
			});
		},
		toAudit({ commentary_id, online_status }) {
			let self = this;
			if (online_status == 0) {
				self.$prompt("请输入下线原因", "请确认是否下线", {
					confirmButtonText: "确认下线",
					cancelButtonText: "取消",
					inputPattern: /\S/,
					inputErrorMessage: "请输入下线原因"
				}).then(({ value }) => {
					updateMovieCommentary({
						offline_reason: value,
						online_status: 1,
						commentary_id: commentary_id
					}).then(res => {
						if (res.code == "000000") {
							this.$message.success("操作成功");
							this.$refs.dataTable.backendPagingFunc();
						} else {
							this.$message.error(res.message);
						}
					});
				});
			} else {
				self.$msgbox({
					title: "请确认是否上线",
					showCancelButton: true,
					confirmButtonText: "确认上线",
					cancelButtonText: "取消"
				}).then(action => {
					updateMovieCommentary({
						offline_reason: "",
						online_status: 0,
						commentary_id: commentary_id,
						user_name: this.user_name
					}).then(res => {
						if (res.code == "000000") {
							this.$message.success("操作成功");
							this.$refs.dataTable.backendPagingFunc();
						} else {
							this.$message.error(res.message);
						}
					});
				});
			}
		},
		loadMoreMovie() {
			let { movieLoading, loadAll } = this.movieSelect;
			if (movieLoading || loadAll) {
				return;
			}
			this.toGetMovieList("", false);
		},
		toGetMovieList(movie_name, flag) {
			if (flag == undefined) {
				this.$set(this.movieSelect, "movie_name", movie_name);
			}
			if (this.movieLoading) {
				return;
			}
			if (movie_name != "") {
				this.$set(this.movieSelect, "page_number", 1);
				this.$set(this.movieSelect, "movieList", []);
				this.$set(this.movieSelect, "loadAll", false);
			}
			if (this.movieSelect.movie_name == "") {
				this.$set(this.movieSelect, "page_number", 1);
				this.$set(this.movieSelect, "movieList", []);
				this.$set(this.movieSelect, "loadAll", false);
				return;
			}
			this.$set(this.movieSelect, "movieLoading", true);
			getMovieList({
				...this.movieSelect
			}).then(res => {
				let { page_number, movieList, page_size } = this.movieSelect;
				if (res.total_size > 0) {
					if (page_number == 1) {
						this.$set(this.movieSelect, "movieList", res.movie_list);
					} else {
						let newDate = [...movieList, ...res.movie_list];
						let map = new Map();
						for (let item of newDate) {
							if (!map.has(item.movie_id)) {
								map.set(item.movie_id, item);
							}
						}
						this.$set(this.movieSelect, "movieList", [...map.values()]);
					}
					if (res.movie_list.length < page_size) {
						this.$set(this.movieSelect, "loadAll", true);
					}
					this.$set(this.movieSelect, "page_number", page_number + 1);
				}
				this.$set(this.movieSelect, "movieLoading", false);
			});
		},
		toGetOfflineList({ commentary_id }) {
			getOfflineRecords({
				commentary_id
			}).then(res => {
				if (res.code == 0) {
					this.offlineList = res.content;
					this.visibleDialog = true;
				}
			});
		}
	},
	watch: {
		$route(to, from) {
			if (to.path.indexOf("/cms/commentary") >= 0) {
				// this.$refs.dataTable.handleChange(true)
				this.$refs.dataTable.backendPagingFunc();
			}
		}
	}
};
</script>
