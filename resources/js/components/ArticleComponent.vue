<template>
    <div class="container">
                 
       <div class="modal fade" id="addArticleModal" tabindex="-1" role="dialog" aria-labelledby="myModalLabel" aria-hidden="true">
  <div class="modal-dialog" role="document">
    <div class="modal-content">
      <div class="modal-header text-center">
        <h4 class="modal-title w-100 font-weight-bold">Nouvel Article</h4>
        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
          <span aria-hidden="true">&times;</span>
        </button>
      </div>
      
      <div class="modal-body mx-3">
        <div class="md-form mb-5">
          <label data-error="wrong" data-success="right" for="form3">Titre:</label>
          <input type="text" id="form3" class="form-control validate" name="title" v-model="article.title" required>
        </div>

        <div class="md-form mb-4">
          <label for="exampleFormControlTextarea1">Article:</label>
    <textarea class="form-control" id="exampleFormControlTextarea1" rows="3" name="body" v-model="article.body"></textarea>
        </div>

      </div>
      <div class="modal-footer d-flex justify-content-center">
        <button class="btn btn-primary" @click="createArticle">Enregistrer <i class="fas fa-paper-plane-o ml-1"></i></button>
      </div>
    </div>
  </div>
</div>

<div class="text-center">
  <a href="" class="btn btn-success btn-rounded mb-4" data-toggle="modal" data-target="#addArticleModal" >Ajouter Article</a>
</div> 
      
    <div class="SC">
      <div class="Post" style="padding-bottom: 40px;" v-for="article in articles" :key="article.id">
        <div class="PostHead">
          <div class="PostTime"> <strong >{{article.article_date}}</strong> </div> 
          <button v-if="article.user_id == user" type="button" class="btn btn-danger btn-sm float-right" @click="deleteArticle(article.id)">Supprimer</button>

          <h2><a href="#">{{article.title}} </a></h2>
          <small class="PostAuthor">Auteur : </small> <small class="PostCat"> <a href="#">{{article.user.name}}</a></small> </div>
        <div class="PostContent">
          <p>{{article.body}}</p>
        </div>
        <div class="PostCom" >
          <img src="images/com.png" alt="">
          <!-- {{article.comments}} -->
          <span>{{article.comments.length}} Comments</span>
          <div v-if="article.comments.length">
          <div class="comment" v-for="comment in article.comments" :key="comment.id" >
            <small>Ecrit par:</small><strong>{{comment.user_id}}</strong>
            <small class="float-right"> {{comment.comment_date}}</small>
            <p>{{comment.content}}</p>
          </div>
          </div>
         <form @submit.prevent="addComment(article.id)">
            <div class="col-sm-10">
            <input class="form-control form-control-sm" type="text"  v-model="new_comment.content" placeholder="votre commentaire ici ..." style="background:#d8fbef4a;">
            <button>add</button>
            </div>
         </form>
        </div>
      </div>
      <div class="clearer"></div>
      
    
 
     
    </div>
        
    </div>
</template>


<script>
let id=0;
    export default {
        data() {
            return {
                delete_article: {},
                delete_article_index: null,
                errors: [],
                user:document.querySelector("meta[name='user-id']").getAttribute('content'),
                article: {
                    title: '',
                    body: '',
                    article_date:'',
                    
                },
                new_comment: {
                    content: '',
                   
                    
                },
                articles: [],
                comments: [],
                update_article: {},
                
            }
        },

        mounted() {
            this.readArticles();
        },

        methods: {
            closeModal() {
                $(".modal").removeClass("is-active");
            },
            currentDateTime() {
              const now = new Date();
              const date = now.getFullYear()+'-'+(now.getMonth()+1)+'-'+now.getDate();
              const time = now.getHours() + ":" + now.getMinutes() + ":" + now.getSeconds();
              return date +' '+ time;
               
            },

            createArticle() {
                axios.post('/article', {
                    title: this.article.title,
                    body: this.article.body,
                    article_date:this.currentDateTime(),
                    user_id: document.querySelector("meta[name='user-id']").getAttribute('content'),
                })
                .then(response => {
                    this.reset();
                    this.articles.push(response.data.article);

                    this.closeModal();
                })
                // .catch(error => {
                //     this.errors = [];
                //     if (error.response.data.errors.title) {
                //         this.errors.push(error.response.data.errors.title[0]);
                //     }

                //     if (error.response.data.errors.body) {
                //         this.errors.push(error.response.data.errors.body[0]);
                //     }
                // });
            },
            addComment(art_id) {
              
                axios.post('/comment', {
                    content: this.new_comment.content,
                    comment_date:this.currentDateTime(),
                    user_id: document.querySelector("meta[name='user-id']").getAttribute('content'),
                    article_id:art_id,
                })
                .then(response => {
                    this.reset();
                    this.comments.push({
                      id:id,
                      value:response.data.new_comment});

                  
                });
                id++;
                // .catch(error => {
                //     this.errors = [];
                //     if (error.response.data.errors.title) {
                //         this.errors.push(error.response.data.errors.title[0]);
                //     }

                //     if (error.response.data.errors.body) {
                //         this.errors.push(error.response.data.errors.body[0]);
                //     }
                // });
            },

            deleteArticle(id) {
                axios.delete('/article/' + id)
                    .then(response => {
                        this.articles.splice(this.delete_article_index, 1);
                        this.closeModal();
                    })
                    .catch(error => {

                    });
            },

            initAddArticle() {
                $("#addArticleModal").addClass("is-active");
            },

            initUpdate(index) {
                this.errors = [];
                $("#updateArticleModal").addClass("is-active");
                this.update_article = this.articles[index];
            },

            readArticles() {
                axios.get('/article')
                    .then(response => {
                        this.articles = response.data.articles;
                    });
            },

            reset() {
                this.article.title = '';
                this.article.body = '';
                this.new_comment.content = '';
                
            },

            updateArticle() {
                axios.patch('/article/' + this.update_article.id, {
                    title: this.update_article.title,
                    body: this.update_article.body,
                    article_date:this.currentDateTime(),
                    user_id: document.querySelector("meta[name='user-id']").getAttribute('content'),
                })
                .then(response => {
                    this.closeModal();
                })
                .catch(error => {
                    this.errors = [];
                    if (error.response.data.errors.title) {
                        this.errors.push(error.response.data.errors.title[0]);
                    }

                    if (error.response.data.errors.body) {
                        this.errors.push(error.response.data.errors.body[0]);
                    }
                });
            },
        }
    }
</script>
