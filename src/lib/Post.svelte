<!-- A post. Profile pictures not appearing while not logged in is intentional. -->

<script>
	import Container from "../lib/Container.svelte";
	import PFP from "../lib/PFP.svelte";
	import FormattedDate from "./FormattedDate.svelte";

	import {profileData, user, ulist} from "../lib/stores.js";
	import * as clm from "../lib/clmanager.js";
	
	import {onMount} from "svelte";
	export let post = {};

	/**
	 * Initialize this post's user profile - gets profile info from the cache or fetches it.
	 */
	function initPostUser() {
		if (!post.user) return;
		if (!($user.name)) return;

		const userName = post.user;

		/**
		 * Fetch the user profile and store it in the cache.
		 */
		const getProfile = () => {
			let _profileData = $profileData;
			_profileData[userName] = {
				pfp_data: -1,
			};
			profileData.set(_profileData);

			clm.meowerRequest({
				cmd: "direct",
				val: {
					cmd: "get_profile",
					val: userName,
				},
				listener: "get_profile_" + userName,
			}).then(val => {
				// Ding dong! The data has arrived.
				_profileData[userName] = val.payload;
				profileData.set(_profileData);
			}).catch(e => {
				// Uh oh - something has gone wrong.
				_profileData[userName] = {
					error: true,
					pfp_data: -2,
					temporary: true,
				};
				profileData.set(_profileData);
			})
		}

		// Do we have a stored profile?
		const _profileData = $profileData;
		if (_profileData[userName]) {
			// Reuse the cached data if the profile isn't temporary
			if (_profileData[userName].temporary) {
				getProfile();
			}
		} else {
			// Get the profile!
			getProfile();
		}
	};
	onMount(initPostUser);
</script>

<Container>
	{#if post.ad}
		<marquee><h2>Advertisement</h2>
		<p>Don't want ads in your WorserMeower feed? Purchase WorserMeower Premium for 0% off at https://worsermeower.bettermeower.app/premium.</p>
		<a href="https://www.youtube.com/watch?v=8ybW48rKBME"><img src={"/ads/ad-"+Math.ceil(Math.random()*4)+".png"} alt="ad" style="min-width: 100%; min-height: 100%;"></a>
		<iframe id="BetterAd" src="https://adservice.bettermeower.app/generate" style="border: medium; width: 728px; height: 90px; overflow: hidden;" scrolling="no"></iframe>
<iframe id="BetterAd" src="https://adservice.bettermeower.app/generate" style="border: medium; width: 728px; height: 90px; overflow: hidden;" scrolling="no"></iframe>
<iframe id="BetterAd" src="https://adservice.bettermeower.app/generate" style="border: medium; width: 728px; height: 90px; overflow: hidden;" scrolling="no"></iframe></marquee>
	{:else}
		<div class="post-header">
			{#if $user.name}
				<div class="pfp">
					<marquee><PFP
						icon={$profileData[post.user] ? $profileData[post.user].pfp_data : -1}
						alt="{post.user}'s profile picture"
						online={false}
					></PFP><marquee>
				</div>
			{/if}
			<div class="creator">
				<h2 class="creator">{post.user}</h2>
				<span class="date">You cannot view dates</span>
			</div>
		</div>
					<marquee><p>{@html post.content}</p></marquee>
	{/if}
</Container>

<style>
	.date {
		font-style: italic;
		font-family: Simvoni-Italic, sans-serif;
		text-decoration: 2px underline dotted;
	}
	.pfp {
		margin-right: 0.2em;
	}
	.post-header {
		display: flex;
		align-items: center;
		flex-wrap: wrap;
	}
	.creator {
		display: inline;
		max-width: 100%;
		margin-left: 0.5em;
	}
	.creator h2 {
		display: block;
		font-size: 200%;
		margin: 0;
	}
</style>
