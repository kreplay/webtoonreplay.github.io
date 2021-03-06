<template>
	<div class="col-md-10 col-md-offset-1">
		<div class="block block-bordered block-themed block-full" :class="classBlock">
			<slot></slot>
		</div>
	</div>
</template>

<script>
	export default {
		data() {
			return {
				chapters: [],
				current_chapter: 1,
				current_page: 1,
				is_loading: false
			};
		},
		props: {
			dataManga: { type: String, required: true },
			dataChapter: { type: Number, default: 1 },
			dataPage: { type: Number, default: 1 },
			dataTimeout: { type: Number, default: 15000 },
			dataSource: { type: String, required: true },
			dataMethod: { type: String, default: "get" }
		},
		computed: {
			classBlock() {
				return (this.is_loading)?['block-opt-refresh']:[];
			},
			methodName() {
				return this.dataMethod.toUpperCase();
			}
		},
		methods: {
			changeLoading(data) {
				this.is_loading = data.state;
			},
			refresh() {
				//Get chapter and pages

				var data = {
					manga_id: this.dataManga
				};
				var that = this;

				switch (this.methodName) {
					case 'GET':
						this.$http.get(this.dataSource, {
							params: data,
							timeout: this.dataTimeout
						}).then(that.onSuccess, that.onFailed);
					break;
					case 'POST':
						this.$http.post(this.dataSource, data, {
							timeout: this.dataTimeout
						}).then(that.onSuccess, that.onFailed);
					break;
				}
			},
			refreshPage() {
				//Broadcast to control
				this.$broadcast('refresh-page', {
					chapters: this.chapters,
					chapter_num: this.current_chapter,
					page_num: this.current_page
				});
			},
			onSuccess(response) {
				//On Success get data from server
				var res = response.data;
				var that = this;

				if (res.success) {
					var chapters = res.chapters;

					that.chapters = [];
					chapters.forEach(function (value, index) {
						var chapter = {
							chapter_num: value.chapter_num,
							chapter_title: value.chapter_title,
							pages: value.pages.slice()
						};

						that.chapters.push(chapter);
					});

					that.refreshPage();
				} else {
					var code = "No Chapter Found";
					var msg = res.message;

					bus.$emit('alert-show', {title: code, text: msg, type: 'error'});
				}
			},
			onFailed(response) {
				//On Failed get data from server
				var code = response.status.toString();
				var msg = response.statusText;

				bus.$emit('alert-show', {title: code, text: msg, type: 'error'});
			},
			setPage(data) {
				this.$broadcast('set-page', data);
			},
			prevPage(e) {
				this.$broadcast('prev-page', e);
			},
			nextPage(e) {
				this.$broadcast('next-page', e);
			}
		},
		created() {
			this.$catch('prev-page', this.prevPage);
			this.$catch('next-page', this.nextPage);
			this.$catch('set-page', this.setPage);
			this.$catch('loading', this.changeLoading);
		},
		mounted() {
			this.current_chapter = this.dataChapter;
			this.current_page = this.dataPage;
			this.refresh();
		}
	}
</script>
