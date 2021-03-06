<template>
	<div
		class="rooms-container app-border-r"
		:class="{ 'rooms-container-full': isMobile }"
		v-show="showRoomsList"
	>
		<slot name="rooms-header"></slot>

		<div class="box-search">
			<div class="icon-search" v-if="rooms.length">
				<svg-icon name="search" />
			</div>
			<input
				type="search"
				:placeholder="textMessages.SEARCH"
				autocomplete="off"
				@input="searchRoom"
				v-show="rooms.length"
			/>
			<div v-if="showAddRoom" class="svg-button add-icon" @click="addRoom">
				<svg-icon name="add" />
			</div>
		</div>

		<loader :show="loadingRooms"></loader>

		<div v-if="!loadingRooms && !rooms.length" class="rooms-empty">
			{{ textMessages.ROOMS_EMPTY }}
		</div>

		<div v-if="!loadingRooms" class="room-list">
			<div
				class="room-item"
				v-for="room in filteredRooms"
				:key="room.roomId"
				:class="{ 'room-selected': selectedRoomId === room.roomId }"
				@click="openRoom(room)"
			>
				<div
					v-if="room.avatar"
					class="room-avatar"
					:style="{ 'background-image': `url('${room.avatar}')` }"
				></div>
				<div class="name-container text-ellipsis">
					<div class="title-container">
						<div
							v-if="userStatus(room)"
							class="state-circle"
							:class="{ 'state-online': userStatus(room) === 'online' }"
						></div>
						<div class="room-name text-ellipsis">
							{{ room.roomName }}
						</div>
						<div v-if="room.lastMessage" class="text-date">
							{{ room.lastMessage.timestamp }}
						</div>
					</div>
					<div
						class="text-last"
						:class="{ 'message-new': room.lastMessage && room.lastMessage.new }"
					>
						<span v-if="room.lastMessage && room.lastMessage.seen">
							<svg-icon name="checkmark" class="icon-check" />
						</span>
						<format-message
							v-if="room.lastMessage"
							:content="getLastMessage(room)"
							:deleted="!!room.lastMessage.deleted"
							:formatLinks="false"
							:textFormatting="textFormatting"
							:singleLine="true"
						></format-message>
						<div v-if="room.unreadCount" class="room-badge">
							{{ room.unreadCount }}
						</div>
					</div>
				</div>
			</div>
		</div>
	</div>
</template>

<script>
import Loader from './Loader'
import SvgIcon from './SvgIcon'
import FormatMessage from './FormatMessage'

import filteredUsers from '../utils/filterItems'

export default {
	name: 'rooms-list',
	components: { Loader, SvgIcon, FormatMessage },

	props: {
		currentUserId: { type: [String, Number], required: true },
		textMessages: { type: Object, required: true },
		showRoomsList: { type: Boolean, required: true },
		showAddRoom: { type: Boolean, required: true },
		textFormatting: { type: Boolean, required: true },
		isMobile: { type: Boolean, required: true },
		rooms: { type: Array, required: true },
		loadingRooms: { type: Boolean, required: true },
		room: { type: Object, required: true }
	},

	data() {
		return {
			filteredRooms: this.rooms || [],
			selectedRoomId: ''
		}
	},

	watch: {
		rooms(val) {
			this.filteredRooms = val
		},
		room(val) {
			if (val) this.selectedRoomId = val.roomId
		}
	},

	methods: {
		searchRoom(ev) {
			this.filteredRooms = filteredUsers(
				this.rooms,
				'roomName',
				ev.target.value
			)
		},
		openRoom(room) {
			if (room.roomId === this.room.roomId && !this.isMobile) return
			this.selectedRoomId = room.roomId
			this.$emit('fetchRoom', { room })
		},
		addRoom() {
			this.$emit('addRoom')
		},
		userStatus(room) {
			if (!room.users || room.users.length !== 2) return

			const user = room.users.find(u => u._id !== this.currentUserId)

			if (user.status) return user.status.state
		},
		getLastMessage(room) {
			const content = room.lastMessage.deleted
				? this.textMessages.MESSAGE_DELETED
				: room.lastMessage.content

			if (room.users.length <= 2) {
				return content
			}

			const user = room.users.find(
				user => user._id === room.lastMessage.sender_id
			)

			if (user._id === this.currentUserId) {
				return content
			}

			return `${user.username} - ${content}`
		}
	}
}
</script>

<style lang="scss" scoped>
.rooms-container {
	display: flex;
	flex-flow: column;
	flex: 0 0 25%;
	min-width: 260px;
	max-width: 500px;
	position: relative;
	background: var(--chat-sidemenu-bg-color);
	height: 100%;
	border-top-left-radius: var(--chat-container-border-radius);
	border-bottom-left-radius: var(--chat-container-border-radius);
}

.rooms-container-full {
	flex: 0 0 100%;
	max-width: 100%;
}

.box-search {
	position: sticky;
	display: flex;
	align-items: center;
	height: 64px;
	padding: 0 15px;
}

.icon-search {
	display: flex;
	position: absolute;
	left: 30px;

	svg {
		width: 18px;
		height: 18px;
	}
}

.add-icon {
	margin-left: auto;
	padding-left: 10px;
}

input {
	height: 38px;
	width: 100%;
	background: var(--chat-bg-color-input);
	color: var(--chat-color);
	border-radius: 4px;
	font-size: 15px;
	outline: 0;
	caret-color: var(--chat-color-caret);
	padding: 10px 10px 10px 40px;
	border: 1px solid var(--chat-sidemenu-border-color-search);
	border-radius: 20px;

	&::placeholder {
		color: var(--chat-color-placeholder);
	}
}

.rooms-empty {
	font-size: 14px;
	color: #9ca6af;
	font-style: italic;
	text-align: center;
	margin: 40px 0;
	line-height: 20px;
	white-space: pre-line;
}

.room-list {
	flex: 1;
	position: relative;
	max-width: 100%;
	cursor: pointer;
	padding: 0 10px 5px;
	overflow-y: auto;
}

.room-item {
	border-radius: 8px;
	align-items: center;
	display: flex;
	flex: 1 1 100%;
	margin-bottom: 5px;
	padding: 0 16px;
	position: relative;
	min-height: 71px;

	&:hover {
		background: var(--chat-sidemenu-bg-color-hover);
		transition: background-color 0.3s cubic-bezier(0.25, 0.8, 0.5, 1);
	}

	&:not(:hover) {
		transition: background-color 0.3s cubic-bezier(0.25, 0.8, 0.5, 1);
	}
}

.room-selected {
	color: var(--chat-sidemenu-color-active) !important;
	background: var(--chat-sidemenu-bg-color-active) !important;

	&:hover {
		background: var(--chat-sidemenu-bg-color-active) !important;
	}
}

.name-container {
	flex: 1;
}

.title-container {
	display: flex;
	align-items: center;
	line-height: 25px;
}

.text-ellipsis {
	white-space: nowrap;
	overflow: hidden;
	text-overflow: ellipsis;
}

.room-name {
	flex: 1;
	color: var(--chat-room-color-username);
	font-weight: 500;
}

.text-last {
	display: flex;
	align-items: center;
	font-size: 12px;
	line-height: 19px;
	color: var(--chat-room-color-message);
}

.message-new {
	color: var(--chat-room-color-username);
	font-weight: 500;
}

.text-date {
	margin-left: 5px;
	font-size: 11px;
	color: var(--chat-room-color-timestamp);
}

.icon-check {
	display: flex;
	height: 14px;
	width: 14px;
	margin-top: -2px;
	margin-right: 2px;
}

.state-circle {
	width: 9px;
	height: 9px;
	border-radius: 50%;
	background-color: var(--chat-room-color-offline);
	margin-right: 6px;
	transition: 0.3s;
}

.state-online {
	background-color: var(--chat-room-color-online);
}

.room-badge {
	background-color: var(--chat-room-bg-color-badge);
	color: var(--chat-room-color-badge);
	font-size: 11px;
	font-weight: 500;
	height: 13px;
	width: auto;
	min-width: 13px;
	border-radius: 50%;
	display: flex;
	align-items: center;
	justify-content: center;
	padding: 3px;
}

@media only screen and (max-width: 768px) {
	.box-search {
		height: 58px;
	}

	.room-list {
		padding: 0 7px 5px;
	}

	.room-item {
		min-height: 60px;
		padding: 0 8px;
	}
}
</style>
