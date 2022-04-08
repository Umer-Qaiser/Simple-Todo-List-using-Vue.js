<template>
    <notifications class="mt-3 ms-3"
                   :duration="2000"
                   :width="250"
                   animation-name="v-fade-left"
                   position="top left" />
    <div class="container my-4">
        <h4 class="text-center">Todo List</h4>
        <h6 class="text-center">An Interview Assignment/Task App.</h6>
        <h6 class="text-center"><b>Created For: </b><a href="https://softsquare.io" target="_blank" alt="SoftSquare Website">SoftSquare </a> &nbsp;<b> Created By: </b><a href="https://www.linkedin.com/in/umerqaiser" target="_blank" alt="Umer Qaiser">Umer Qaiser</a></h6>
        <hr />

        <div class="row d-flex justify-content-center">
            <div class="col-xs-12 col-lg-6 my-3">
                <div class="form-group mb-3">
                    <label for="todo" class="form-label">Add a todo item:</label>
                    <div class="row">
                        <div class="col-10">
                            <input v-model="todo"
                                   type="text"
                                   class="form-control"
                                   name="todo"
                                   id="todo"
                                   placeholder="For example: Meeting with the CTO."
                                   v-bind:class="{ 'is-invalid': input_errors.length > 0, 'is-valid': input_errors.length == 0 && todo != '' }" />
                            <div class="invalid-feedback">
                                <span :key="key" v-for="(error,key) in input_errors">{{ error }}</span>
                            </div>
                        </div>
                        <div class="col-2 d-grid gap-2">
                            <button class="btn btn-primary btn-s float-end" @click="save">Add</button>
                        </div>
                    </div>
                </div>
                <div class="form-group"></div>
            </div>
        </div>

        <div class="row d-flex justify-content-center mt-2" v-if="ToDos.length">
            <div class="col-md-6">
                <h5 class="mb-3">All Your Todos:</h5>
                <div style="list-style-type: none;">
    <li v-for="(todo,index) in ToDos"
        v-bind:key="index"
        class="row flex justify-content-center align-items-center py-1">
        <div class="col-8">
            <span v-bind:style="todo.done ? 'text-decoration:line-through;' : ''">{{ todo.todo }}</span>
        </div>
        <div class="col-2 d-flex justify-content-center">
            <span class="form-check form-switch"
                  @click="done_todo(todo)"
                  style="margin-left:20px;">
                <input class="form-check-input"
                       type="checkbox"
                       role="switch"
                       id="flexSwitchCheckDefault"
                       data-onstyle="#1f2023"
                       :checked="todo.done" />
            </span>
        </div>
        <div class="col-2 d-flex justify-content-center">
            <button class="btn btn-danger btn-sm" @click="delete_todo(todo.id)">Delete</button>
        </div>
    </li>
    </div>
    </div>
    </div>
    <hr />
    <div class="row">
        <div class="col-lg-12 text-center">
            <img class="py-3 px-3 text-center" src="../assets/logo-softsquare.png" alt="SoftSquare"
                 style="background-color: #333; width: 200px;" />
        </div>
    </div>
    </div>
</template>

<script>
    export default {
        data() {
            return {
                todo: "",
                ToDos: [],
                errors: [],
                newTodo: {
                    todo: "",
                    done: false
                },
                responseData: null,
                input_errors: []
            };
        },
        mounted() {
            this.getTodos();
        },
        methods: {
            async getTodos() {
                try {
                    const response = await this.$axios.get("/todos");
                    this.ToDos = response.data;
                } catch (error) {
                    this.errors.push(error);
                }
            },
            async save() {
                if (this.input_errors.length > 0 || this.todo == '') {
                    if (this.todo == '' && this.input_errors.length == 0)
                        this.input_errors.push('ToDo field cannot be left blank')
                    this.input_errors.forEach((value) => {
                        this.$notify(value);
                    });
                } else {
                    try {
                        this.newTodo.todo = this.todo;
                        const response = await this.$axios.post("/todos", this.newTodo);
                        this.responseData = response.data;
                    } catch (error) {
                        this.errors.push(error);
                    }
                    this.getTodos();
                    this.$notify("Added Succesfully");
                    this.todo = "";
                }
            },
            async delete_todo(index) {
                try {
                    const response = await this.$axios.delete("/todos/" + index);
                    this.responseData = response.data;
                } catch (error) {
                    this.errors.push(error);
                }
                this.getTodos();
                this.$notify("Deleted Succesfully.");
            },
            async done_todo(todo) {
                try {
                    const response = await this.$axios.put("/todos/" + todo.id, { "todo": todo.todo, "done": !todo.done });
                    this.responseData = response.data;
                } catch (error) {
                    this.errors.push(error);
                }
                this.getTodos();
                this.$notify("Updated Successfully.");
            },
        },
        watch: {
            todo(val) {
                this.input_errors = [];

                if (val.length < 3 || val.length > 40) {
                    this.input_errors.push('ToDo field be Minimum 6, Maximum 25 characters.')
                    return;
                }
            }
        }
    };
</script>