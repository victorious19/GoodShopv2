<template>
<form class="content m-3" @submit.prevent="create" v-if="categories">
    <div class="row">
        <div class="col-md-12">
            <div class="card card-primary">
                <div class="card-body">
                    <div class="form-group">
                        <label for="inputTitle">Title</label>
                        <input type="text" id="inputTitle" class="form-control" v-model="title" v-bind:class="{'is-invalid':errors.title}">
                        <div class="invalid-feedback" v-if="errors.title">{{errors.title[0]}}</div>
                    </div>
                    <div class="form-group">
                        <label for="inputDescription">Description</label>
                        <textarea id="inputDescription" class="form-control" rows="4" v-model="description" v-bind:class="{'is-invalid':errors.description}"></textarea>
                        <div class="invalid-feedback" v-if="errors.description">{{errors.description[0]}}</div>
                    </div>
                    <div class="form-group">
                        <label for="inputCategory">Category</label>
                        <select id="inputCategory" class="form-control custom-select" v-model="category">
                            <option v-for="category in categories" :value="category.id">{{ category.name }}</option>
                        </select>
                    </div>
                    <div class="form-group">
                        <label for="inputPrice">Price</label>
                        <input type="number" id="inputPrice" class="form-control" step="any" v-model="price" v-bind:class="{'is-invalid':errors.price}">
                        <div class="invalid-feedback" v-if="errors.price">{{errors.price[0]}}</div>
                    </div>
                    <div class="form-group">
                        <label for="inputPicture">Picture</label>
                        <input type="file" id="inputPicture" multiple class="form-control" accept=".png, .jpg" @change.prevent="upload" v-bind:class="{'is-invalid':errors.picture}">
                        <div class="invalid-feedback" v-if="errors.picture">{{errors.picture[0]}}</div>
                    </div>
                    <div class = "alert alert-danger" v-if="errors.main">
                        <a class="close" data-dismiss="alert"> × </a>
                        <strong> Error! </strong> Server error. <br>{{errors.main}}
                    </div>
                </div>
            </div>
            <!-- /.card -->
        </div>
    </div>
    <div class="row">
        <div class="col-12">
            <router-link to="/profile/products" class="btn btn-secondary">Cancel</router-link>
            <input type="submit" value="Create new Product" class="btn btn-success float-right">
        </div>
    </div>
</form>
<Spinner v-else></Spinner>
</template>

<script>
import Spinner from "./Spinner";
export default {
    name: "CreateProduct",
    components: {
        Spinner
    },
    data() {
        return {
            title: '',
            description: '',
            category: '',
            price: null,
            formData: new FormData(),
            errors: {main: ''},
            categories: []
        }
    },
    mounted() {
        axios.get('/api/categories', {})
        .then(res => {
            this.categories = res.data
        })
        .catch(err => {
            alert(err)
        })
    },
    methods: {
        upload(event) {
            let files = event.target.files
            for (let i = 0; files[i]; i++) {
                this.formData.append(`pictures[${i}]`, files[i])
            }
        },
        create() {
            this.errors = {main: ''}
            this.formData.append('title', this.title)
            this.formData.append('description', this.description)
            this.formData.append('price', this.price)
            this.formData.append('category_id', this.category)
            axios.defaults.headers['Authorization'] = `Bearer ${this.$cookie.get('token')}`;
            axios.post('/api/products', this.formData)
            .then(res => {
                this.$router.push({path:'/profile/products', query: {class:'alert-primary', content:'Successful creation'}})
            })
            .catch(err => {
                console.log(err)
                if (err.response.data.errors) this.errors = err.response.data.errors
                else if(err.response.data.message) this.errors.main =  err.response.data.message
                else this.errors.main =  err.response.statusText
            })
        }
    }
}
</script>

<style scoped>

</style>
