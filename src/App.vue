<template>
	<div class="container">
		<h1>{{ title }}</h1>

		<!-- инпут + кнопка -->
		<form 
			class="form" 
			@submit.prevent="addTask"
		>
			<input 
				class="form_input" 
				:class="{form_input__error: error}" 
				placeholder="Новая заметка" 
				v-model="newTask" 
				ref="inputField"
			/>

			<div 
				v-if="error" 
				class="form_error"
			>
				{{ error }}
			</div>
			
			<button 
				class="form_add-btn" 
				type="submit" 
				:disabled="loading"
			>
				{{ loading ? 'Загрузка' : 'Добавить' }}
			</button>
		</form>

		<!-- фильтр -->
		<div class="filters">
			<button 
				v-for="filter in filters" 
				:key="filter.value" 
				class="filters_btn"
				:class="{ active: currentFilter === filter.value }" 
				@click="currentFilter = filter.value"
			>
				{{ filter.label }}
			</button>
		</div>

		<!-- лист тасок -->
		<TransitionGroup 
			name="task-list" 
			tag="div" 
			class="tasks"
		>
			<TodoItem 
				v-for="task in filteredTasks" 
				:key="task.id" 
				:task="task" 
				@check="toggleTask" 
				@edit="editTask"
				@delete="deleteTask" 
			/>
		</TransitionGroup>

		<!-- очистить все таски -->
		<button 
			v-if="tasks.length > 0 && currentFilter === 'all'" 
			@click="clearAll" 
			class="clear-button"
		>
			Очистить все заметки
		</button>

		<!-- заглушка на емпти таски -->
		<div 
			v-if="filteredTasks.length === 0" 
			class="empty"
		>
			<img src="./assets/img/empty.png"></img>
			<p>Ничего...</p>
		</div>
	</div>
</template>

<script setup lang="ts">
import { ref, onMounted, watch, computed, Ref } from 'vue'
import TodoItem from './components/ToDoItem.vue'

interface Task {
	id: number;
	text: string;
	completed: boolean;
}

const title: string = 'To-Do List'
const newTask = ref<string>('')
const inputField = ref<HTMLInputElement | null>(null)
const error = ref<string>('')

const loading = ref<boolean>(false)
const tasks = ref<Task[]>([])

const currentFilter = ref<'all' | 'active' | 'completed'>('all')

const filters: { label: string; value: 'all' | 'active' | 'completed' }[] = [
	{ label: 'Все', value: 'all' },
	{ label: 'Активные', value: 'active' },
	{ label: 'Выполненные', value: 'completed' }
]

const filteredTasks = computed(() => {
	switch (currentFilter.value) {
		case 'active':
			return tasks.value.filter(task => !task.completed)
		case 'completed':
			return tasks.value.filter(task => task.completed)
		default:
			return tasks.value
	}
})

onMounted(() => {
	inputField.value?.focus()

	const savedTasks = localStorage.getItem('tasks')
	if (savedTasks) {
		tasks.value = JSON.parse(savedTasks) as Task[]
	}
})

watch(
	tasks,
	(newTasks) => {
		localStorage.setItem('tasks', JSON.stringify(newTasks))
	},
	{ deep: true }
)

const addTask = async (): Promise<void> => {
	if (!newTask.value.trim()) {
		error.value = 'Пусто...'
		return
	}

	loading.value = true
	error.value = ''

	await new Promise(resolve => setTimeout(resolve, 500))

	tasks.value.push({
		id: Date.now(),
		text: newTask.value,
		completed: false
	})

	newTask.value = ''
	loading.value = false
}

const toggleTask = (id: number): void => {
	const task = tasks.value.find(task => task.id === id)
	if (task) task.completed = !task.completed
}

const deleteTask = (id: number): void => {
	tasks.value = tasks.value.filter(task => task.id !== id)
}

const editTask = (updatedTask: Task): void => {
	const task = tasks.value.find(task => task.id === updatedTask.id)
	if (task) {
		task.text = updatedTask.text
	}
}

const clearAll = (): void => {
	if (confirm('Точно удаляем все заметки?')) {
		tasks.value = []
	}
}
</script>


<style lang="scss">
.container {
	padding: 40px 20px;
	margin: 0 auto;
	max-width: 600px;
}

h1 {
	font-size: 32px;
	font-weight: 600;
	line-height: 150%;
	text-align: center;
	text-transform: uppercase;
	color: $input-text;

	margin-bottom: 30px;
}

.form {
	width: 100%;
	display: flex;
	justify-content: space-between;
	flex-wrap: wrap;

	@media(max-width: 460px) {
		display: block;
	}

	&_input {
		width: calc(100% - 136px);
		box-sizing: border-box;
		padding: 6px 16px;
		color: $input-text;

		border: 2px solid $input-border;
		border-radius: 6px;

		order: 0;

		@media(max-width: 460px) {
			width: 100%;
			margin-bottom: 8px;
		}

		&::placeholder {
			font-size: 16px;
			line-height: 100%;
			color: $input-placeholder;
		}

		&:hover {
			border-color: $input-hover-border;
		}

		&:focus-visible {
			border-color: $input-focus-border;
		}

		&__error {
			border-color: $error;

			&:hover {
				border-color: $error;
			}

			&:focus-visible {
				border-color: $error;
			}
		}
	}

	&_add-btn {
		min-width: 110px;
		padding: 6px 16px;
		color: $button-text;
		font-size: 16px;
		text-align: center;
		background-color: $button-bg;
		border: 2px solid $button-border;
		border-radius: 6px;

		order: 2;

		cursor: pointer;

		&:hover {
			background-color: $button-hover-bg;
			border-color: $button-hover-border;
		}

		&:focus-visible {
			background-color: $button-focus-bg;
			background-color: $button-focus-border;
		}

		@media(max-width: 460px) {
			width: 100%;
		}
	}

	&_error {
		width: 100%;
		padding-left: 18px;
		margin-bottom: 16px;
		color: $error;
		font-size: 12px;

		order: 3;
}
}

.filters {
	box-sizing: border-box;
	display: flex;
	gap: 10px;
	margin-top: 30px;
	margin-bottom: 20px;
	justify-content: left;

	&_btn {
		padding: 6px 16px;
		font-size: 16px;
		color: $input-text;
		text-align: center;
		border: 2px solid $input-border;
		border-radius: 6px;
		cursor: pointer;

		&.active {
			background-color: $button-bg;
			border-color: $button-hover-border;
			color: white;
		}

		&:focus-visible {
			background-color: $button-hover-bg;
			border-color: $button-hover-bg;
			color: white;
		}

		&:hover {
			background-color: $button-hover-bg;
			border-color: $button-hover-border;
			color: white;
		}
	}

	@media(max-width: 390px) {
		flex-wrap: wrap;
		gap: 0;

		&>button {
			width: 100%;
			margin-bottom: 4px;
		}
	}
}

.clear-button {
	margin-top: 30px;
	padding: 6px 16px;
	color: $button-text;
	font-size: 16px;
	text-align: center;
	background-color: $button-bg;
	border: 2px solid $button-border;
	border-radius: 6px;

	cursor: pointer;

	&:hover {
		background-color: $button-hover-bg;
		border-color: $button-hover-border;
	}

	&:focus-visible {
		background-color: $button-focus-bg;
		background-color: $button-focus-border;
	}
}

.task-list {

	&-enter-active,
	&-leave-active {
		transition: all 0.5s ease;
	}

	&-enter-from,
	&-leave-to {
		opacity: 0;
		transform: translateY(30px);
	}

	&-move {
		transition: transform 0.3s ease;
	}
}

.empty {
	margin-top: 60px;
	display: flex;
	justify-content: center;
	flex-wrap: wrap;

	& img {
		max-width: 300px;
		margin-bottom: 16px;
	}

	p {
		width: 100%;
		text-align: center;
	}
}
</style>