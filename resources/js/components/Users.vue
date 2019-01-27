<template>
    <div class="container">
        <div class="row mt-5">
            <div class="col-md-12">
                <div class="card">
                    <div class="card-header">
                        <h3 class="card-title">Users List</h3>
                        <div class="card-tools">
                            <button type="submit" @click="openModal()" class="btn btn-success">Add User
                                <i class="fa fa-user-plus fa-fw"></i></button>
                        </div>
                    </div>
                    <div class="card-body table-responsive p-0">
                        <table class="table table-hover">
                            <tbody>
                                <tr>
                                    <th>ID</th>
                                    <th>Name</th>
                                    <th>Email</th>
                                    <th>Type</th>
                                    <th>Registered at</th>
                                    <th>Action</th>
                                </tr>
                                <tr v-for="(user, index) in users" :key="user.id">
                                    <td>{{user.id}}</td>
                                    <td>{{user.name}}</td>
                                    <td>{{user.email}}</td>
                                    <td>{{user.type | upText}}</td>
                                    <td>{{user.created_at | myDate}}</td>
                                    <td><a href="#" @click="editModal(user)"><i class="fa fa-edit"></i></a> || <a
                                            href="#" v-on:click="deleteUser(user.id, index)"><i class="fa fa-trash red"></i></a></td>
                                </tr>
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>
        </div>
        <div class="modal fade" id="usersModal" tabindex="-1" role="dialog" aria-labelledby="modal-title"
             aria-hidden="true">
            <div class="modal-dialog modal-dialog-centered" role="document">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title" id="modal-title" v-show="!editMode">Add User</h5>
                        <h5 class="modal-title" id="modal-title" v-show="editMode">Update User's Data</h5>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                            <span aria-hidden="true">&times;</span>
                        </button>
                    </div>
                    <form @submit.prevent="editMode ? updateUser() : createUser()">
                        <div class="modal-body">
                            <div class="form-group">
                                <input v-model="form.name" type="text" name="name"
                                       placeholder="Enter name"
                                       class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
                                <has-error :form="form" field="name"></has-error>
                            </div>
                            <div class="form-group">
                                <input v-model="form.email" type="text" name="email"
                                       placeholder="Enter email"
                                       class="form-control" :class="{ 'is-invalid': form.errors.has('email') }">
                                <has-error :form="form" field="email"></has-error>
                            </div>
                            <div class="form-group">
                                <input v-model="form.password" type="password" name="password"
                                       placeholder="Enter password"
                                       class="form-control" :class="{ 'is-invalid': form.errors.has('password') }">
                                <has-error :form="form" field="password"></has-error>
                            </div>
                            <div class="form-group">
                                <textarea v-model="form.bio" type="text" name="bio"
                                       placeholder="Bio"
                                          class="form-control" :class="{ 'is-invalid': form.errors.has('bio') }"></textarea>
                                <has-error :form="form" field="bio"></has-error>
                            </div>
                            <div class="form-group">
                                <select v-model="form.type" name="type"
                                          class="form-control" :class="{ 'is-invalid': form.errors.has('type') }">
                                    <option value="" selected>Select User Role</option>
                                    <option value="admin">Admin</option>
                                    <option value="user">User</option>
                                    <option value="author">Author</option>
                                </select>
                                <has-error :form="form" field="type"></has-error>
                            </div>
                        </div>
                        <div class="modal-footer">
                            <button type="button" class="btn btn-danger" data-dismiss="modal" v-on:click="testSweet()">
                                Close</button>
                            <button type="submit" class="btn btn-primary" v-show="!editMode">Create</button>
                            <button type="submit" class="btn btn-success" v-show="editMode">Update</button>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        data() {
            return {
                editMode: false,
                users: [],
                form: new Form({
                    id: '',
                    name: '',
                    email: '',
                    'password': '',
                    type: '',
                    bio: '',
                    photo: ''
                })
            }
        },
        methods: {
            editModal(user) {
                this.editMode = true;
                this.form.reset();
                $('#usersModal').modal('show');
                this.form.fill(user);
            },
            openModal() {
                this.editMode = false;
                this.form.reset();
                $('#usersModal').modal('show')                   
            },
            loadUsers() {
                axios.get('/api/users').then(response => {
                    this.users = response.data.data
                })
            },
            createUser() {
                this.$Progress.start()
                this.form.post('/api/users').then(response => {                   
                    this.users.unshift(response.data);
                    $('#usersModal').modal('hide')                   
                });
                
                               
                this.$Progress.finish()
            },
            deleteUser(id, index) {
                console.log('user deeted');
                this.form.delete('/api/users/' + id).then(response => {
                    this.users.splice(index, 1);
                }).catch(errors => {
                    console.log(errors)
                }) 
            },
            updateUser() {
                console.log('update user');
                this.form.put('/api/users/'+this.form.id)
                .then(response => {
                    var ind = this.users.findIndex(x => x.id == this.form.id)
                    Vue.set(this.users, ind, response.data)
                    $('#usersModal').modal('hide') 
                })
                .catch(() => {

                })
            }
        },
        created() {
            this.loadUsers()
        }
    }
</script>
