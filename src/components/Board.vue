<template>
  <div class="comment_warpper">
    <left-nav />
  <div class="board_inputting">
      <div class="comment_inputting_box">
              <quill-editor 
                    v-model="comment" 
                    ref="myQuillEditor" 
                    :options="editorOption" 
                    @blur="onEditorBlur($event)" @focus="onEditorFocus($event)"
                    @change="onEditorChange($event)">
            </quill-editor>
          <div class="submit_reset_btn">
               <span v-if="isReplyState">
                    <!-- <h3>你将要回复{{ comments[replyObj] ? comments[replyObj].username : replyTo }}, 请输入内容:</h3> -->
                    <Button type="success" @click="submitReply(logged_user)">确认回复</Button>
                    <Button type="error" @click="cancelReply(logged_user)">取消回复</Button>
              </span>
               <span v-else>
                  <Button type="success" @click="submitComment(logged_user)">留言</Button>
                  <Button type="error" @click="resetBoard">重置</Button>
               </span>
              
          </div>
      </div>
  </div>
  <div v-for="(item,index) in comments" :key="item._id" class="comment_item">
        <div  class="avatar">
           <Avatar :style="randomColor(index)" size="small" icon="ios-person" ></Avatar>
        </div>
        <div class="comment_description"><h4>{{comments.length -index}}楼</h4></div>
        <div class="comment">
          <div v-html="item.comment" class="comment-core"></div>
          <ul>
            <li><Icon type="ios-man-outline" /> {{item.username ? item.username + '留言' : ''}}</li>
            <li><Icon type="ios-time-outline" /> {{item.commentDate.substring(0,10)}}</li>
            <li @click="replyBoardComment(item,index)"><Icon type="ios-create-outline" /> 回复</li>
            <li @click="updateThumbDown(item,index)"><Icon type="ios-thumbs-down" /> 差评:{{ item.dislikes ? item.dislikes : 0 }}</li>
            <li @click="updateThumbUp(item,index)"><Icon type="ios-thumbs-up" /> 点赞:{{ item.likes ? item.likes : 0 }}</li>
          </ul>
            <div class="reply" v-for="reply in item.commentReplys">
                <strong><span v-html="reply.comment"></span> </strong>  
                 <span class="reply-nav">{{ reply.from }}@ {{ reply.to }} ##{{ reply.time ?  reply.time.toString().substring(0,10) : '' }}</span>
                <span class="reply-nav">
                    <li @click="replyComment( item,index,reply.from)"><Icon type="ios-create-outline" /> 回复 {{ reply.from }}</li>
                </span>
            </div>
        </div>
        <hr />
  </div>
  <Footer v-if="footerShowing" class="foot-nav"></Footer>
  </div>
</template>

<script>

// import Editor from '@tinymce/tinymce-vue'
import Footer from './foot-nav/Footer.vue'
import { quillEditor } from 'vue-quill-editor'
import 'quill/dist/quill.core.css'
import 'quill/dist/quill.snow.css'
import 'quill/dist/quill.bubble.css'
import { mapState, mapActions } from 'vuex'
import LeftNav from './nav/LeftNav'

export default {
  name: 'board',
  data () {
    return {
      valueText: '',
      comment: '想说点什么呢..',
      comments: [],
      footerShowing: false,
      isReplyState: false,
      replyObj : -1,
      replyTo: '',
      editorOption:{

      }
    }
  },
  components:{
      Footer,
      // editor: Editor,
      quillEditor,
      LeftNav
  },
  mounted(){
      this.enableLoading()
      this.loadingData()
      this.disableLoading()
  },
  computed:{
    ...mapState([
      'logged_user'
    ])
  },
  methods:{
      ...mapActions([
        'enableLoading',
        'disableLoading'
      ]),
      async loadingData(){
          await this.$axios.get('/api/getAllBoardComments').then(res => {
            if(res.status === 200 && res.data.status == 'true'){
                for(let comment of res.data.comments){
                     this.comments.push(comment)
                }
                this.footerShowing = true
            }
        }).catch(err => {
        })
      },
      commentInputedChanged(){
          // console.log(this.comment)
      },
      contentChanged(){

      },
      submitComment(logged_user){
        // console.log( this.comment)
        const username = logged_user || '匿名'
        const comment = this.comment
        if(username == '匿名'){
          this.$Message.warning('你将要匿名留言吗?');
          this.$Modal.confirm({
                      title: '确定进行匿名留言么？',
                      content: '<p>可取消并返回主页注册登录后非匿名留言</p><p></p>',
                      onOk: () => {
                         if(comment){
                                this.footerShowing = false
                                this.$axios.post('/api/add_board_comment',{ username, comment, avatar: '匿名',commentDate: new Date(),  likes: 0, dislikes: 0,
                                    }).then(res => {
                                      // console.log(res.data.status)
                                        if(res.data.status === 'true'){
                                              this.$axios.get('/api/getAllBoardComments/').then(res => {
                                                  if(res.status === 200 && res.data.status === 'true'){
                                                  // console.log(res.data.comments)
                                                    this.comment = ''
                                                    this.comments.splice(0,this.comments.length)
                                                    for(let comment of res.data.comments){
                                                        this.comments.push(comment)
                                                      }
                                                      this.footerShowing = true
                                                  }
                                              }).catch(err => {

                                              })
                                        }
                                    }).catch(err => {
                                })
                            }

                      },
                      onCancel: () => {     
                        return       
                      }
          });
        }else{
              if(comment){
                    this.footerShowing = false
                    this.$axios.post('/api/add_board_comment',{ username, comment, avatar: '匿名',commentDate: new Date(),  likes: 0, dislikes: 0,
                        }).then(res => {
                          // console.log(res.data.status)
                            if(res.data.status === 'true'){
                                  this.$axios.get('/api/getAllBoardComments/').then(res => {
                                      if(res.status === 200 && res.data.status === 'true'){
                                      // console.log(res.data.comments)
                                        this.comment = ''
                                        this.comments.splice(0,this.comments.length)
                                        for(let comment of res.data.comments){
                                            this.comments.push(comment)
                                          }
                                          this.footerShowing = true
                                      }
                                  }).catch(err => {

                                  })
                            }
                        }).catch(err => {
                    })
                }
        } 
      },
      resetBoard(){
          this.comment = ''
      },
      randomColor(index){
          let R = Math.floor(Math.random() * 255);
          let G = Math.floor(Math.random() * 255);
          let B = Math.floor(Math.random() * 255);
          return {
            background: 'rgb(' + R + ',' + G + ',' + B + ')'
          }
      },
      replyBoardComment(item,index){
          this.isReplyState = true
          this.replyObj = index
          this.replyTo = item.username
          this.comment= "@" + item.username + ":"
      },
     submitReply(logged_user){
          let index = this.replyObj
          let _id = this.comments[index]._id
          let from  = logged_user
          let time  = new Date()
          let to = this.replyTo || this.comments[index].username
          let comment = this.comment
          if(from && to && comment){
            this.$axios.post('/api/add_board_reply',{ from, to, _id, time, comment }).then(res => {
                console.log(res.data.status)
                if(res.data.status === 'ture'){
                    this.cancelReply()
                }
            }).catch(err => {

            })
          }
     },
     cancelReply(logged_user){
          this.isReplyState = false
          this.comment = ""
     } ,
    replyComment(item,index,to){
          this.replyTo = to
          this.isReplyState = true
          this.replyObj = index
          this.comment= "@" + to  + ":"
          // console.log(to)
          
    },
    onEditorBlur(){//失去焦点事件

    },
    onEditorFocus(){//获得焦点事件

    },
    onEditorChange(){//内容改变事件
        //  console.log(this.content)
    },
      updateThumbDown(item,index){
          // const {_id, dislikes } = item
          let _id = item._id || 0
          let dislikes = item.dislikes || 0
          this.$axios.post('/api/board/update_comment_dislikes',{ _id , dislikes }).then(res => {
                  if(res.data.status == 'true'){
                        // console.log(res.data.dislikes)
                        this.comments[index].dislikes = res.data.dislikes
                  }
          }).catch(err => {

          })
      },
      updateThumbUp(item,index){
          // const {_id, likes } = item
          let  _id = item._id || 0
          let likes = item.likes || 0
          this.$axios.post('/api/board/update_comment_likes',{ _id, likes }).then(res => {
                  if(res.data.status == 'true'){
                        // console.log(res.data.likes)
                        this.comments[index].likes = res.data.likes
                  }
          }).catch(err => {

          })
      },


  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>

@media only screen and (min-width:540px){

.comment_warpper{
        margin: 0 auto;
        background: url("../assets/wbg.jpg") repeat;
        z-index: 1;
        .board_inputting{
			max-width: 60%;
			width: 1000px;
			margin: 0 auto;
            position: sticky;
            top: 0;
            z-index: 10;
            .comment_inputting_box{
                margin-top: 1rem;
                font-size: 0.8rem;
                .submit_reset_btn{
                  padding-top: 1rem;
                  margin-bottom: 5rem;
                }
            }
    }
    .comment_item{
        // width: 60%;
        // margin: 0 10% 0 20%;
		max-width: 60%;
		width: 1000px;
        margin: 1rem auto 0.8rem;
        padding: 1rem 1rem 0 0;
        // min-height: 10vh;  
        overflow: hidden;
         .avatar{
              float: left;
              height: 2rem;
              width: 2rem;
              display: inline-block;
              display: flex;
              justify-content: center;
              align-items: center;
          } 
          .comment_description{
                h4{
                  line-height: 2rem;
                }
          }  
          .comment{
              padding-bottom: 0.4rem;
              margin-left: 0.5rem;
              .comment-core{
                padding: 1rem 0;
                font-size: 1rem;
                line-height: 2rem;
              }
              .comment-core >>> p{

                }
              ul {
                   li{
                        list-style-type: none;
                        display: inline-block;
                        margin: 0 0.3rem;
                        padding: 0 0.2rem;
                        font-size: 0.8rem;
                        min-width: 6rem;
                        text-align: center;
                        &:hover{
                          cursor: pointer;
                          text-decoration: underline;
                        }
                    }
              }
              .reply{
                 font-family:Arial, Helvetica, sans-serif;
                 font-size: 0.8rem;
                 padding: 0.5rem 0 0.6rem 2rem;
                 background: rgba(212, 182, 182, 0.1);
                 .reply-nav{
                      font-size: 0.6rem;
                      line-height: 1em;
                      color: blue;
                      li{
                        list-style-type: none;
                        display: inline;
                        &:hover{
                            transform: scale(1.25);
                            cursor: pointer;
                        }
                      }
                 }
              }
          }
    }
}

}

@media only screen and (max-width:540px){
.comment_warpper{
    .board_inputting{
        position: sticky;
        top: 0;
        z-index: 10;
        .comment_inputting_box{
            margin-top: 1rem;
            font-size: 0.6rem;
            .submit_reset_btn{
              padding-top: 1rem;
            }
        }
    }
    .comment_item{
        padding: 1rem 1rem 0 0;
        min-height: 15vh;  
        overflow: hidden;
         .avatar{
              float: left;
              height: 2rem;
              width: 2rem;
              display: inline-block;
              display: flex;
              justify-content: center;
              align-items: center;
          } 
          .comment_description{
                h4{
                  font-size: 0.8rem;
                  line-height: 2rem;
                }
          }  
          .comment{
              padding-bottom: 0.4rem;
              margin-left: 0.5rem;
              .comment-core{
                padding: 1rem 0;
                font-size: 0.8rem;
                line-height: 2rem;
              }
              ul {
                   li{
                        list-style-type: none;
                        display: inline-block;
                        font-size: 0.8rem;
                        text-align: center;
                        font-size: 0.8rem;
                        &:hover{
                          cursor: pointer;
                          text-decoration: underline;
                        }
                    }
              }
              .reply{
                 font-family:Arial, Helvetica, sans-serif;
                 font-size: 0.8rem;
                 padding: 1rem ;
                 background: rgba(212, 182, 182, 0.1);
                 .reply-nav{
                   font-size: 0.6rem;
                   line-height: 1em;
                   color: blue;
                 }
              }
          }

    }
    .foot-nav{
      font-size: 0.6rem
    }
}

}




</style>
