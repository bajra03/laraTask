<template>
    <div class="container">
        <div class="row justify-content-center">
            <div class="col-md-8">
                <div class="card">
                    <div class="card-header">
                        <button @click="showModal()" class="btn btn-primary btn-sm float-right">+ Task</button>
                        Task List
                    </div>

                    <div class="card-body">
                        <table class="table table-bordered table-striped">
                            <thead>
                                <tr>
                                    <th>#</th>
                                    <th>Task Title</th>
                                    <th>Level</th>
                                    <th></th>
                                </tr>
                            </thead>
                            <tbody>
                                <tr v-for="(task, index) in tasks" :key="index">
                                    <td>{{ index + 1 }}</td>
                                    <td>{{ task.title }}</td>
                                    <td>{{ task.prior }}</td>
                                    <td>
                                        <button @click="updateTask(task)" class="btn btn-success btn-sm">Edit</button>
                                        <button @click="deleteTask(task)" class="btn btn-danger btn-sm">Delete</button>
                                    </td>
                                </tr>
                            </tbody>
                        </table>
                    </div>

                    <div class="card-footer">
                        <ul class="pagination">
                            <li v-bind:class="[{disabled: !pagination.prev_page_url}]" class="page-item"><a @click="getTasks(pagination.prev_page_url)" class="page-link" href="#">Previous</a></li>
                            <li class="page-item disabled"><a class="page-link" href="#">Page {{ pagination.current_page }} of {{ pagination.last_page }}</a></li>
                            <li v-bind:class="[{disabled: !pagination.next_page_url}]" class="page-item"><a @click="getTasks(pagination.next_page_url)" class="page-link" href="#">Next</a></li>
                        </ul>
                    </div>
                </div>
            </div>
        </div>
        <!-- Modal -->
        <div class="modal fade" id="modal-form" tabindex="-1" role="dialog" aria-labelledby="exampleModalLabel" aria-hidden="true">
            <div class="modal-dialog" role="document">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title" id="exampleModalLabel">Task Form</h5>
                        <button @click="resetModal()" type="button" class="close" data-dismiss="modal" aria-label="Close">
                            <span aria-hidden="true">&times;</span>
                        </button>
                    </div>
                    <div class="modal-body">
                        <div v-if="errors.length > 0" class="alert alert-danger" role="alert">
                            <ul>
                                <li v-for="(error, index) in errors" :key="index">
                                    {{ error }}
                                </li>
                            </ul>
                        </div>
                        <div class="form">
                            <label for="title">Title</label>
                            <input v-model="task.title" type="text" name="title" class="form-control">
                        </div>

                        <div class="form">
                            <label for="prior">Prior</label>
                            <select v-model="task.prior" name="prior" id="" class="form-control">
                                <option value="low">Low</option>
                                <option value="medium">Medium</option>
                                <option value="high">High</option>
                            </select>
                        </div>
                    </div>
                    <div class="modal-footer">
                        <button @click="saveTask()" type="button" class="btn btn-primary">Save</button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    data() {
        return{
            tasks: [],
            task: {
                id: '',
                title: '',
                prior: ''
            },
            errors: [],
            edit: false,
            pagination: {}
        }
    },
    methods:{
        showModal() {
            this.errors = [];
            $("#modal-form").modal("show");
        },

        getTasks(page_url) {
            let url = page_url || '/task';
            axios.get(url)
                .then(response => {
                    this.tasks = response.data.data;
                    this.createPagination(response.data.meta, response.data.links);
                });
        },

        saveTask() {
            if (this.edit === false) {
                axios.post('/task', {
                    title: this.task.title,
                    prior: this.task.prior
                })
                .then(response => {
                    this.getTasks();
                    this.resetModal();
                    $("#modal-form").modal("hide");
                })
                .catch(error => {
                    this.errors =[];
                    if(error.response.data.errors.title) {
                        this.errors.push(error.response.data.errors.title[0]);
                    }
                    if(error.response.data.errors.prior) {
                        this.errors.push(error.response.data.errors.prior[0]);
                    }
                })
            } else {
                axios.patch('/task/' + this.task.id, {
                    title: this.task.title,
                    prior: this.task.prior
                })
                .then(response => {
                    this.getTasks();
                    this.resetModal();
                    $("#modal-form").modal("hide");
                })
                .catch(error => {
                    this.errors =[];
                    if(error.response.data.errors.title) {
                        this.errors.push(error.response.data.errors.title[0]);
                    }
                    if(error.response.data.errors.prior) {
                        this.errors.push(error.response.data.errors.prior[0]);
                    }
                })
            }
        },

        updateTask(task) {
            this.task.id = task.id;
            this.task.title = task.title;
            this.task.prior = task.prior;

            this.edit = true;

            this.showModal();
        },

        deleteTask(task) {
            let decision = confirm('Are you sure to delete this data?');
            if (decision === true) {
                axios.delete('/task/' + task.id)
                .then(response => {
                    this.getTasks();
                })
                .catch(error => {
                    console.log(error)
                })
            }
        },

        resetModal() {
            this.task.title = '';
            this.task.prior = '';
            this.edit = false;
        },

        createPagination(meta, links) {
            let pagination = {
                current_page: meta.current_page,
                last_page: meta.last_page,
                next_page_url: links.next,
                prev_page_url: links.prev
            }

            this.pagination = pagination;
        }
    },
    mounted() {
        this.getTasks();
    }
}
</script>

<style>

</style>
