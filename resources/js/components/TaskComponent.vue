<template>
    <div class="container">
        <div class="row justify-content-center">
            <div class="col-md-8">
                <div class="card">
                    <div class="card-header">
                        <button @click="showModal()" class="btn btn-primary btn-sm float-right">
                            + Add Task
                        </button>
                        Task List</div>
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
                                        <button @click="updateTask(task)" class="btn btn-info btn-sm">Edit</button>
                                        <button @click="deleteTask(task)" class="btn btn-danger btn-sm">Delete</button>
                                    </td>
                                </tr>
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>
        </div>
        <!-- Modal -->
        <div class="modal fade" id="modal-form" tabindex="-1" role="dialog" aria-labelledby="exampleModalLabel" aria-hidden="true">
            <div class="modal-dialog" role="document">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title" id="exampleModalLabel">Modal title</h5>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                        <span aria-hidden="true">&times;</span>
                        </button>
                </div>
            <div class="modal-body">
                <div v-if="(errors.length > 0)" class="alert alert-danger" role="alert">
                    <ul>
                        <li v-for="(error, index) in errors" :key="index">
                            {{ error }}
                        </li>
                    </ul>
                </div>
                <div class="form-group">
                    <label for="">Task Title</label>
                    <input v-model="task.title" type="text" name="title" id="" class="form-control">
                </div>
                <div class="form-group">
                    <label for="">Prior</label>
                    <select v-model="task.prior" name="prior" id="" class="form-control">
                        <option value="low">Low</option>
                        <option value="medium">Medium</option>
                        <option value="high">High</option>
                    </select>
                </div>
            </div>
            <div class="modal-footer">
                <button @click="resetModal()" type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
                <button @click="saveTask()" type="button" class="btn btn-primary">Submits</button>
            </div>
            </div>
        </div>
        </div>
    </div>
</template>

<script>
export default {
    data(){
        return{
            tasks : [],
            task:{
                id : '',
                title : '',
                prior : ''
            },
            errors:[],
            edit:false
        }
    },
    methods : {
        showModal(){
            this.errors = [];
            $("#modal-form").modal("show");
        },

        getTask(){
            axios.get('/task')
                .then(response => {
                    this.tasks = response.data.data;
                    // console.log(this.tasks);
                });
        },

        saveTask(){
            if (this.edit === false) {
                    axios.post('/task',{
                    title: this.task.title,
                    prior: this.task.prior
                })
                .then(response =>{
                    this.getTask();
                    $("#modal-form").modal("hide");
                    this.resetModal();
                })
                .catch(error => {
                    this.errors = [];
                    if (error.response.data.errors.title) {
                            this.errors.push(error.response.data.errors.title[0]);
                    }
                    if (error.response.data.errors.prior) {
                            this.errors.push(error.response.data.errors.prior[0]);
                    }
                })
            }else{
                axios.put('/task/' + this.task.id, {
                    title: this.task.title,
                    prior: this.task.prior
                })
                .then(response =>{
                    this.getTask();
                    $("#modal-form").modal("hide");
                    this.resetModal();
                })
                .catch(error => {
                    this.errors = [];
                    if (error.response.data.errors.title) {
                            this.errors.push(error.response.data.errors.title[0]);
                    }
                    if (error.response.data.errors.prior) {
                            this.errors.push(error.response.data.errors.prior[0]);
                    }
                })
            }
        },

        resetModal(){
            this.task.title = '';
            this.task.prior = '';
            this.edit = false;
        },

        updateTask(task){
            this.task.id = task.id;
            this.task.title = task.title;
            this.task.prior = task.prior;

            this.edit = true;

            this.showModal();
        },

        deleteTask(task){
            let decision = confirm('apakah anda yakin?');
            if(decision === true){
                axios.delete('/task/' + task.id)
                .then(response => {
                    this.getTask();
                })
                .catch(error => {
                    console.log(error);
                })
            }
        }


    },
     mounted() {
            this.getTask();
        }
}
</script>
