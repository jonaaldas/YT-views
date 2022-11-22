<template>
	<div class="flex flex-col items-center">
		<h1 class="text-2xl">Get YT videos by view count</h1>
		<input
			v-model="channelID"
			placeholder="Youtube Channel ID"
		/>
		<button
			class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
			@click="getYtData()"
		>
			Get YT Data
		</button>
		<button
			v-show="showTable === true"
			class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
			@click="exportTableToExcel()"
			id="sheetjsexport"
		>
			<b>Export as XLSX</b>
		</button>

		<table
			v-show="showTable === true"
			class="table-auto w-full"
			id="TableToExport"
		>
			<thead class="bg-gray-50">
				<th class="px-6 py-2 text-xs text-gray-500">N</th>
				<th class="px-6 py-2 text-xs text-gray-500">Channel Name</th>
				<th class="px-6 py-2 text-xs text-gray-500">Video Title</th>
				<th class="px-6 py-2 text-xs text-gray-500">Link</th>
				<th class="px-6 py-2 text-xs text-gray-500">Views</th>
				<th class="px-6 py-2 text-xs text-gray-500">tags</th>
			</thead>
			<tbody class="bg-white">
				<tr
					class="whitespace-nowrap"
					v-for="(video, index) of videos"
					:key="index"
				>
					<td class="px-6 py-4 text-sm text-gray-500">{{ index + 1 }}</td>
					<td class="px-6 py-4 text-sm text-gray-500">
						{{ video.channelName }}
					</td>
					<td class="px-6 py-4 text-sm text-gray-500">
						{{ video.videoTitle }}
					</td>
					<td class="px-6 py-4 text-sm text-gray-500">
						<a
							target="_blank"
							:href="video.link"
							>{{ video.link }}</a
						>
					</td>
					<td class="px-6 py-4 text-sm text-gray-500">{{ video.views }}</td>
					<td class="flex items-center flex-wrap justify-center gap-2 p-px">
						<div
							v-for="(tag, index) of video.tags"
							:key="index"
							class="text-xs font-bold leading-sm uppercase px-3 py-1 bg-blue-200 text-blue-700 rounded-full"
						>
							{{ tag }}
						</div>
					</td>
				</tr>
			</tbody>
		</table>
	</div>
</template>

<script>
	// @ts-nocheck
	import axios from 'axios';
	import Papa from 'papaparse';
	export default {
		data() {
			return {
				channelID: '',
				API_KEY: 'AIzaSyAXk38eRmUfn0HvW_h-NzDoPdTmr5IfLYs',
				videos: [],
				showTable: false,
			};
		},
		methods: {
			async getYtData() {
				if (this.channelID === '') {
					console.log('Missing Channel ID');
				} else {
					const res = await axios.get(
						`https://www.googleapis.com/youtube/v3/search?part=snippet&channelId=${this.channelID}&key=${this.API_KEY}&maxResults=20&order=viewcount`
					);
					let obj = res.data.items.map((eachVideo) => {
						return {
							videoID: eachVideo.id.videoId,
							link: `https://www.youtube.com/watch?v=${eachVideo.id.videoId}`,
							channelName: eachVideo.snippet.channelTitle,
							videoTitle: eachVideo.snippet.title,
							thumbNail: eachVideo.snippet.thumbnails.default,
							views: '',
							tags: '',
						};
					});

					for (const index in obj) {
						const res = await axios.get(
							`https://youtube.googleapis.com/youtube/v3/videos?part=statistics&id=${obj[index].videoID}&key=${this.API_KEY}`
						);
						const tagRes = await axios.get(
							` https://youtube.googleapis.com/youtube/v3/videos?part=snippet&id=${obj[index].videoID}&key=${this.API_KEY}`
						);
						obj[index].views = res.data.items[0].statistics.viewCount;
						obj[index].tags = tagRes.data.items[0].snippet.tags;
					}
					this.showTable = true;
					this.videos = obj;
				}
			},
			exportTableToExcel() {
				/* Create worksheet from HTML DOM TABLE */
				var wb = XLSX.utils.table_to_book(
					document.getElementById('TableToExport')
				);
				/* Export to file (start a download) */
				XLSX.writeFile(wb, 'SheetJSTable.xlsx');
			},
		},
	};
</script>
<style></style>
