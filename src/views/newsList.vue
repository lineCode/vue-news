<template>
    <div class="news-list row">
        <div class="col-md-12" style="padding-left: 0;padding-right: 0">
            <p style="color:#666565">{{channelTitle}}</p>
            <div v-infinite-scroll="loadMore" infinite-scroll-disabled="busy"  infinite-scroll-distance="10" style="position: relative">
                <b-card style="border: none">
                    <b-list-group v-for="newslList in newslLists">
                        <b-list-group-item v-for="newsContent in newslList">

                            <a class="no-mb" v-bind:href="newsContent.link">{{newsContent.title}}</a>  <!--属性中不要用{{}}-->

                        </b-list-group-item>

                    </b-list-group>
                </b-card>
                <div v-if="page>allPage">
                    没有更多数据了
                </div>
                <loading v-show="loading"></loading>
            </div>

        </div>

    </div>
</template>

<script>
    import { mapGetters } from 'vuex'
    import loading from '../components/loading.vue'
    export default {
        name: 'newsList',
        data () {
            return {
                newsApi:'http://route.showapi.com/109-35',
                page:0,
                allPage:0,
                newslLists:[],
                loading:false,
                channelTitle:'',
                busy: false
            }
        },
        computed: {
                ...mapGetters(['showapi_appid','showapi_sign'])  //取 sore.js里面的值，相当于全局变量
        },
        watch:{
            '$route'(to,from){
                if(to.name=='newsList'){
                    this.newslLists=[];
                    this.getList(true); //切换菜单时 页码初始化
                }
            }
        },
        methods:{
            getList(newChannel){
                if(newChannel){
                    this.page=0;
                }
                this.page++;
                this.busy = true;
                let _this=this;
                this.channelTitle=this.$route.params.channelName;
                //请求发送前 拦截器
                this.axios.interceptors.request.use(function (config) {
                    _this.loading=true;  //打开loading动画
                    return config;
                }, function (error) {
                    return Promise.reject(error);
                });
                this.axios.get(''+_this.newsApi+'?showapi_appid='+this.showapi_appid+'&showapi_sign='+this.showapi_sign+'&channelId='+this.$route.params.id+'&page='+_this.page+'')
                        .then(function(response){
                            if(response.data.showapi_res_body.pagebean.contentlist.length<=0){

                                _this.loading=false;  //关闭动画

                            }else{
                                _this.allPage=response.data.showapi_res_body.pagebean.allPages;//总页数
                                console.log('***'+_this.allPage)
                                //_this.newslLists=response.data.showapi_res_body.pagebean.contentlist;
                                _this.newslLists.push(response.data.showapi_res_body.pagebean.contentlist)
                                _this.loading=false;  //关闭动画

                            }

                        })
                        .catch(function(error){

                        });
                this.busy = false;
            },

            loadMore() {
                if(this.page<=this.allPage){
                    this.getList();
                }
            }
        },
        mounted(){
            //this.getList();
            /*this.$nextTick(function() {
                this.getList();
            })*/
        },
        components:{loading}

    }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>