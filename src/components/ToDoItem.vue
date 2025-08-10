<template>
	<div class="task">
		<label class="custom-checkbox">
			<input type="checkbox" 
				:checked="task.completed" 
				@change="$emit('check', task.id)"
			>
			<span class="checkmark"></span>
		</label>
		<span 
			v-if="!isEditing" 
			:class="{ completed: task.completed }" 
			@dblclick="editTask"
		>
			{{ task.text }}
		</span>

		<input 
			v-else class="task_input" 
			type="text" v-model="editedText" 
			@blur="saveNewTask" 
			@keyup.enter="saveNewTask"
			@keyup.escape="cancelNewChange" 
			v-focus
		>

		<button 
			@click="$emit('delete', task.id)" 
			class="task_delete"
		>
			<img src="../assets/img/dlt.png" alt="delete">
		</button>
	</div>
</template>

<script setup lang="ts">
import { ref } from 'vue'

interface Task {
	id: number;
	text: string;
	completed: boolean;
}

const props = defineProps<{
	task: Task
}>()

const emit = defineEmits<{
	(e: 'check', id: number): void;
	(e: 'delete', id: number): void;
	(e: 'edit', updatedTask: Task): void;
}>()

const isEditing = ref<boolean>(false)
const editedText = ref<string>('')
let shouldSave = true

const editTask = (): void => {
	editedText.value = props.task.text
	isEditing.value = true
	shouldSave = true
}

const saveNewTask = (): void => {
	if (shouldSave && editedText.value.trim()) {
		emit('edit', {
			...props.task,
			text: editedText.value.trim()
		})
	}
	isEditing.value = false
}

const cancelNewChange = (): void => {
	shouldSave = false
	isEditing.value = false
}
</script>


<style lang="scss">
.tasks {
	margin-top: 20px;
}

.task {
	display: flex;
	align-items: center;
	padding: 16px;
	margin-bottom: 20px;
	position: relative;
	border-bottom: 1px solid $input-hover-border;

	& input[type="checkbox"] {
		margin-right: 10px;
	}

	& span {
		max-width: calc(100% - 60px);
		overflow-wrap: break-word;
		white-space: normal;
	}

	& span.completed {
		text-decoration: line-through;
		color: #aaa;
	}

	&_input {
		width: fit-content;
		max-width: calc(100% - 60px);
		border: 0.3px solid $input-hover-border;
		background-color: transparent;
	}

	&_delete {
		position: absolute;
		right: 10px;

		display: flex;
		background: none;
		border: 1px solid rgba(0, 0, 0, 0.258);
		border-radius: 4px;
		cursor: pointer;
		padding: 6px 6px;

		font-size: 16px;
		line-height: 120%;
		color: $button-text;

		&:hover {
			background-color: #ff6b6b;
		}

		&:focus-visible {
			border-color: #ff6b6b;
			background-color: #ff6b6b;
		}

		&>img {
			width: 16px;
			height: 16px;
		}
	}

	.custom-checkbox {
		position: relative;
		display: inline-flex;
		align-items: center;
		min-height: 20px;
		min-width: 20px;
		margin-right: 12px;
		cursor: pointer;

		input {
			position: absolute;
			opacity: 0;
			cursor: pointer;
			height: 0;
			width: 0;

			&:checked~.checkmark {
				background-color: $button-bg;
				border-color: $button-border;

				&:after {
					display: block;
				}
			}

			&:focus~.checkmark {
				border-color: $input-focus-border;
				box-shadow: 0 0 0 2px rgba($input-focus-border, 0.2);
			}

			&:hover:not(:checked)~.checkmark {
				border-color: $input-hover-border;
			}
		}

		.checkmark {
			flex-shrink: 0;
			display: inline-block;
			min-width: 20px;
			height: 20px;
			background-color: $input-bg;
			border: 1px solid $input-border;
			border-radius: 4px;
			transition: all 0.2s ease;
			box-sizing: border-box;

			&:after {
				content: "";
				position: absolute;
				display: none;
				left: 7px;
				top: 3px;
				width: 5px;
				height: 10px;
				border: solid $button-text;
				border-width: 0 2px 2px 0;
				transform: rotate(45deg);
			}
		}

		&:hover input:not(:checked)~.checkmark {
			background-color: darken($input-bg, 5%);
		}

		&.error .checkmark {
			border-color: $error;
		}
	}
}
</style>