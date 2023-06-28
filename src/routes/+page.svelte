<script lang="ts">
	import { Calendar } from '@fullcalendar/core'
	import dayGridPlugin from '@fullcalendar/daygrid'
	import timeGridPlugin from '@fullcalendar/timegrid'
	import listPlugin from '@fullcalendar/list'
	import timelinePlugin from '@fullcalendar/resource-timeline'
	import resource from '@fullcalendar/resource'
	import interactionPlugin from '@fullcalendar/interaction'
	import { Draggable } from '@fullcalendar/interaction'
	import { onMount } from 'svelte'
	import { fly } from 'svelte/transition'

	let calendarEl: HTMLElement
	let eventsContainer: HTMLElement

	let events = [
		{ title: 'SM006', extendedProps: { workOrderId: 1 } },
		{ title: 'SM007', extendedProps: { workOrderId: 2 } },
		{ title: 'SM008', extendedProps: { workOrderId: 3 } }
	]

	onMount(() => {
		new Draggable(eventsContainer, {
			itemSelector: '.event'
		})
		let calendar = new Calendar(calendarEl, {
			headerToolbar: {
				left: 'prev,next today',
				center: 'title',
				right: 'resourceTimelineWeek'
			},
			plugins: [dayGridPlugin, timeGridPlugin, listPlugin, interactionPlugin, timelinePlugin],
			schedulerLicenseKey: 'CC-Attribution-NonCommercial-NoDerivatives',
			resources: [
				{ id: '1', title: 'Derek' },
				{ id: '2', title: 'Joseph' },
				{ id: '3', title: 'Brandon' }
			],
			events: [
				{
					title: 'SM006',
					workOrderId: 1,
					start: '2023-06-28T09:00:00',
					end: '2023-06-28T10:00:00',
					resourceId: '1'
				},
				{
					title: 'SM096',
					workOrderId: 12,
					start: '2023-06-28T09:30:00',
					end: '2023-06-28T11:30:00',
					resourceId: '1'
				}
			],

			editable: true,
			droppable: true, // this allows things to be dropped onto the calendar
			drop: function (info) {
				events.findIndex((event) => event.title === info.draggedEl.innerText)
				events.splice(
					events.findIndex((event) => event.title === info.draggedEl.innerText),
					1
				)
				events = events
			},
			eventReceive: function (info) {
				const fullObject = {
					title: info.event.title,
					workOrderId: info.event.extendedProps.workOrderId,
					start: info.event.start,
					end: info.event.end,
					resourceId: info.event.getResources()[0].id
				}
				console.log('event received', fullObject)
			},
			eventChange: function (info) {
				const fullObject = {
					title: info.event.title,
					workOrderId: info.event.extendedProps.workOrderId,
					start: info.event.start,
					end: info.event.end,
					resourceId: info.event.getResources()[0].id
				}
				console.log('event changed', fullObject)
			},

			slotMinTime: '09:00:00',
			slotMaxTime: '18:00:00',

			eventDragStop: function (info) {
				let elCoords = { x: info.jsEvent.pageX, y: info.jsEvent.pageY }
				let itemsBoundingRect = eventsContainer.getBoundingClientRect()
				// check if bounding rects overlap
				if (
					elCoords.x < itemsBoundingRect.right &&
					elCoords.x > itemsBoundingRect.left &&
					elCoords.y < itemsBoundingRect.bottom &&
					elCoords.y > itemsBoundingRect.top
				) {
					console.log('overlapping adding' + info.event.title)
					events.push({
						title: info.event.title,
						extendedProps: {
							workOrderId: info.event.extendedProps.workOrderId
						}
					})
					events = events
					info.event.remove()
				}
			}
		})
		calendar.render()
	})
</script>

<div
	class="bg-white max-w-xl w-full p-2 rounded-md my-4 shadow-md mx-auto flex gap-1 flex-wrap"
	bind:this={eventsContainer}>
	{#each events as event (event.extendedProps.workOrderId)}
		<div
			in:fly={{ x: 100 }}
			class="event btn variant-filled-primary max-w-[10rem] grow transition-none"
			data-event={JSON.stringify(event)}>
			{event.title}
		</div>
	{/each}
</div>

<div class="bg-white max-w-3xl p-2 rounded-md shadow-md w-full mx-auto">
	<div id="calendar" bind:this={calendarEl} />
</div>
