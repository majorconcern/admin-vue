<template>
	<div class="cl-route-nav">
		<p class="title" v-if="browser.isMini">{{ lastName }}</p>

		<template v-else>
			<el-breadcrumb>
				<el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
				<el-breadcrumb-item v-for="(item, index) in list" :key="index">{{
					(item.meta && item.meta.label) || item.name
				}}</el-breadcrumb-item>
			</el-breadcrumb>
		</template>
	</div>
</template>

<script>
import { mapGetters } from "vuex";
import _ from "lodash";

export default {
	name: "cl-route-nav",

	data() {
		return {
			list: []
		};
	},

	watch: {
		$route: {
			immediate: true,
			handler(route) {
				const deep = item => {
					if (route.path === "/") {
						return false;
					}

					if (item.path == route.path) {
						return item;
					} else {
						if (item.children) {
							const ret = item.children.map(deep).find(Boolean);

							if (ret) {
								return [item, ret];
							} else {
								return false;
							}
						} else {
							return false;
						}
					}
				};

				this.list = _(this.menuGroup)
					.map(deep)
					.filter(Boolean)
					.flattenDeep()
					.value();

				if (this.list.length === 0) {
					this.list.push(route);
				}
			}
		}
	},

	computed: {
		...mapGetters(["menuGroup", "browser"]),

		lastName() {
			return _.last(this.list).name;
		}
	}
};
</script>

<style lang="scss" scoped>
.cl-route-nav {
	white-space: nowrap;

	/deep/.el-breadcrumb {
		&__inner {
			font-size: 13px;
			padding: 0 10px;
			font-weight: normal;
			letter-spacing: 1px;
		}
	}

	.title {
		font-size: 14px;
		font-weight: 500;
		margin-left: 5px;
	}
}
</style>
