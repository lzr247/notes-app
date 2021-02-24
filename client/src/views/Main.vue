<template>
    <div class="main">

        <v-app-bar flat app light>
            <span class='text-uppercase font-weight-bold text-h6'>
                {{this.name}} - Beleške
            </span>
            <v-spacer></v-spacer>
            <v-btn @click="logout" text color='#000'>
                <span>Odjava</span>
                <v-icon right>mdi-exit-to-app</v-icon>
            </v-btn>
        </v-app-bar>
        
        <v-main>
            <v-container>
                <v-row>
                    <v-col cols='12' sm='6'>
                        <h1>Nova beleška</h1>
                        <v-form ref='form' v-model='valid' lazy-validation>
                            <v-text-field
                            v-model='title' 
                            label='Naslov' 
                            solo
                            solo-inverted
                            light
                            :rules="[rules.requiredTitle]"
                            ></v-text-field>
                            <v-textarea
                            v-model='content'
                            solo
                            label="Sadržaj beleške"
                            counter
                            :rules="[rules.requiredContent]"
                            ></v-textarea>
                            <v-btn @click="addNote">
                                Unesi belešku
                            </v-btn>
                        </v-form>
                    </v-col>
                    <v-col cols='12' sm='6'>
                        <h1>Vaše beleške</h1>
                        <v-alert dense type="info" v-if="notesArray.length==0">Niste uneli nijednu belešku.</v-alert>
                        <div v-for="(item,index) in notesArray" :key='index' class="notesDiv mb-4 py-4 d-flex flex-row justify-space-between align-center">
                            <span class="ml-5 pa-4 font-weight-bold" @click="openModalShow(item)" style="cursor:pointer;">{{item.title}}</span> 
                            <div>
                                <v-btn @click="openModalEdit(item)" class='mx-4'>Izmeni</v-btn>
                                <v-btn @click="deleteNote(item._id)" class='rounded-circle mr-5'>X</v-btn>
                            </div>
                        </div>
                    </v-col>
                </v-row>
                <v-snackbar
                    v-model="snackbar"
                    :timeout="timeout"
                    >
                    {{ responseData }}

                    <template v-slot:action="{ attrs }">
                        <v-btn
                        color="#E9C46A"
                        text
                        v-bind="attrs"
                        @click="snackbar = false"
                        >
                        Zatvori
                        </v-btn>
                    </template>
                </v-snackbar>

                <v-dialog v-model="dialogEdit" max-width="400px">
                    <v-card style="overflow:hidden!important;">
                        <v-card-title>Izmena beleške</v-card-title>
                        <v-divider></v-divider>
                        <v-card-actions class="d-flex flex-column justify-center d-sm-flex flex-sm-column justify-sm-space-around">
                            <v-form ref='form2' v-model='valid2' lazy-validation>
                                <v-text-field
                                v-model='titleUpdate' 
                                solo-inverted
                                light
                                background-color="#aaa"
                                flat
                                :rules="[rules.requiredTitle]"
                                ></v-text-field>
                                <v-textarea
                                v-model='contentUpdate'
                                solo
                                label=""
                                :rules="[rules.requiredContent]"
                                ></v-textarea>
                                <v-btn color="#E9C46A" class="black--text mt-3 mt-sm-0 mb-1" @click="editNote()">Izmeni belešku</v-btn>
                            </v-form>
                            <v-btn color="#E9C46A" class="black--text my-3" @click="dialogEdit = false">Nazad</v-btn>
                            
                        </v-card-actions>
                    </v-card>
                </v-dialog>

                <v-dialog v-model="dialogShow" max-width="400px">
                    <v-card style="overflow:hidden!important;">
                        <v-card-title>{{this.modalDataShow.title}}</v-card-title>
                        <v-divider></v-divider>
                        <v-card-actions class="d-flex flex-column justify-center d-sm-flex flex-sm-column justify-sm-space-around">
                        <v-card-text class="text-left my-5 py-5">
                            <pre>
                                {{this.modalDataShow.content}}
                            </pre>  
                        </v-card-text>
                        <v-btn color="#E9C46A" class="black--text my-3" @click="dialogShow = false">Nazad</v-btn>
                            
                        </v-card-actions>
                    </v-card>
                </v-dialog>

            </v-container>
        </v-main>
    </div>
</template>

<script>
import axios from 'axios';
export default {
    name: 'Main',
    data(){
        return{
            contentUpdate: '',
            titleUpdate:'',
            modalDataEdit:{},
            dialogEdit:false,
            modalDataShow:{},
            dialogShow:false,
            timeout: 2000,
            responseData: '',
            snackbar: false,
            name: '',
            id: '',
            valid:true,
            valid2:true,
            title: '',
            content: '',
            rules: {
                requiredTitle: value => !!value || 'Morate uneti naslov beleške.',
                requiredContent: value => !!value || 'Morate uneti sadržaj beleške.'
            },
            notesArray:[]
        }
    },
    created(){
        if(localStorage.getItem('token') === null || localStorage.getItem('token') == ''){
            this.$router.push('/login');
        }
    },
    methods:{
        openModalEdit(data){
            this.modalDataEdit = data;
            this.titleUpdate = this.modalDataEdit.title;
            this.contentUpdate = this.modalDataEdit.content;
            this.dialogEdit = true;
        },
        openModalShow(data){
            this.modalDataShow = data;
            this.modalDataShow.content = this.modalDataShow.content.trim();
            this.dialogShow = true;
        },
        addNote(){
            if(this.$refs.form.validate()){
                console.log(this.content)
                axios 
                .post('/api/notes',{
                    headers: { token: localStorage.getItem('token')},
                    data: {
                        title: this.title,
                        content: this.content
                    }
                })
                .then(res => {
                    this.responseData = res.data[0];
                    this.notesArray = res.data[1];
                    this.snackbar = true;
                })
                .catch( error => {
                    if(error.response){
                        console.log(error.response.data)
                    }
                })
            }
            this.title = '';
            this.content = '';
            this.$refs.form.resetValidation()
        },
        editNote(){
            if(this.$refs.form2.validate()){
                axios
                .put('/api/notes',{
                    headers: { token: localStorage.getItem('token')},
                    data:{
                        id: this.modalDataEdit._id,
                        title: this.titleUpdate,
                        content: this.contentUpdate
                    }
                })
                .then( res => {
                    this.notesArray = res.data;
                    this.dialogEdit = false;
                    this.responseData = 'Uspešna izmena beleške.';
                    this.snackbar = true;
                })
                .catch( error => {
                    if(error.response){
                        console.log(error.response.data)
                    }
                })
            }
        },
        deleteNote(id){
            axios
            .delete('/api/notes', {
                headers: { token: localStorage.getItem('token')},
                data: {
                    id: id
                }
            })
            .then(res => {
                this.notesArray = res.data;
                this.responseData = 'Uspešno brisanje beleške.';
                this.snackbar = true;
            })
            .catch( error => {
                if(error.response){
                    console.log(error.response.data)
                }
            })
        },
        logout(){
            localStorage.clear();
            this.$router.push('/login');
        }
    },
    mounted(){
        axios
        .get('/api/users/me', {
            headers: { token: localStorage.getItem('token')}
        })
        .then( res => {
            this.name = res.data[0].name;
            this.id = res.data[0]._id;
        })
        .catch( error => {
          if(error.response){
            console.log(error.response.data)
          }
        })

        axios
        .get('/api/notes',{
            headers: { token: localStorage.getItem('token')}
        })
        .then( res => {
            this.notesArray = res.data;
        })

    }
};
</script>

<style lang="scss" scoped>
    .main{
        min-height: 100vh;
        background-color:#d9d9d9;
    }
    .notesDiv{
        background-color: #fff;
    }
    #input-9{
        color: black!important;
    }
</style>