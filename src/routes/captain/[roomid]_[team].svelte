<script lang="ts">
	import { io } from "socket.io-client";
	import Header from '$lib/Header.svelte';

	import { page } from '$app/stores';
	import { browser } from '$app/env';

	import MapSelect from '$lib/MapSelect.svelte';
	import Button from '$lib/Button.svelte';

	import { banns, picks, selectedMap, phase, turn } from '../../store';
	import type { Room } from "$types/Room";
	import { User, UserRole } from "../../../types/User";
	import { goto } from "$app/navigation";
	import { Phase, Turn } from '../../../types/Room';

	const team = $page.params.team

	let action : () => void

	if (browser) {
		const socket = io()

		socket.emit('joinRoom', {
			role: team,
			room: $page.params.roomid
		})

		socket.on('ban', (newBanns: string[]) => {
			banns.set(newBanns)
		})
		socket.on('pick', (newPicks: string[]) => {
			picks.set(newPicks)
		})
		socket.on('turn', (newTurn: Turn) => {
			turn.set(newTurn)
		})
		socket.on('phase', (newPhase: Phase) => {
			phase.set(newPhase)
		})
		
		socket.on('roomState', (room: Room) => {
			phase.set(room.phase)
			turn.set(room.turn)
			banns.set(room.banns)
			picks.set(room.picks)
		})

		socket.on('user', (user: User) => {
			if (user === undefined || user.role === UserRole.SPECTATOR) {
				goto("/spectator/" + $page.params.roomid)
			}
		})

		action = () => {
			socket.emit($phase, $selectedMap)
			selectedMap.set(undefined)
		}
	}
</script>

<Header>
	{#if $phase === Phase.NONE}
		Pleas Wait
	{:else if $phase === Phase.SIDE}
		{#if $turn !== team}
			Opponent is picking a Side
		{:else}
			pick a Side
		{/if}
	{:else}
		{#if $turn !== team}
			{`Opponent is ${$phase}ing a Map`}
		{:else}
			{`${$phase} a Map`}
		{/if}
	{/if}
</Header>

<MapSelect disableAll={$turn !== team || $phase === Phase.SIDE} />
<Button on:click={action} disabled={$selectedMap == undefined || $turn !== team} >Confirm</Button>
