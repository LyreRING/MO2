<template>
  <v-container>
    <v-row justify="center">
      <v-col cols="12" lg="7" class="mo2editor">
        <v-skeleton-loader
          v-if="loading"
          v-bind="attrs"
          type="heading, list-item-avatar, paragraph@9"
        ></v-skeleton-loader>
        <div v-if="!loading">
          <div class="mo2title text-break">
            <h1>{{ title }}</h1>
          </div>
          <v-row v-if="authorLoad" class="mb-6">
            <avatar :size="40" :user="author"></avatar>
            <a
              @click="$router.push('/account/' + author.id)"
              class="text--lighten-2 ml-2 mt-2"
              >{{ author.name }}</a
            >
            <span class="ml-2 grey--text mt-2">{{
              blog.entityInfo.createTime.substr(0, 10)
            }}</span>
            <v-spacer />
            <v-tooltip v-if="draft" bottom>
              <template v-slot:activator="{ on, attrs }">
                <v-btn plain small v-bind="attrs" v-on="on">
                  <v-icon>mdi-eye-check</v-icon>
                </v-btn>
              </template>
              <span>This is a draft</span>
            </v-tooltip>
            <v-btn @click="edit" v-if="user.id === blog.authorId" plain small>
              <v-icon>mdi-file-document-edit</v-icon>
            </v-btn>
            <v-btn
              @click="deleteArticle"
              v-if="user.id === blog.authorId"
              plain
              small
            >
              <v-icon>mdi-delete</v-icon>
            </v-btn>
          </v-row>
          <v-row v-else class="mb-6">
            <v-skeleton-loader
              class="col"
              type="list-item-avatar"
            ></v-skeleton-loader>
          </v-row>
          <!-- <img
          class="ma-5"
          src="https://th.bing.com/th/id/OIP.dnWfZl6P-0Pl47j7PhZodQHaHJ?w=187&h=180&c=7&o=5&dpr=2&pid=1.7"
        />
        <v-row justify="center" class="mb-5">• • •</v-row> -->
          <div
            v-html="$sanitize(html)"
            class="mo2content mt-10"
            spellcheck="false"
          ></div>
          <delete-confirm
            :title="'确认删除'"
            :content="deleteContent"
            :show.sync="showDelete"
            @confirm="confirmDelete"
          />
          <div style="padding-bottom: 1rem"></div>
          <v-row v-if="!draft">
            <v-col class="offset-lg-10 offset-9"
              ><v-icon @click="share">mdi-share</v-icon></v-col
            >
            <v-col class=""
              ><v-icon @click="loadComment">mdi-message-reply-outline</v-icon
              >{{ commentNum }}</v-col
            >
          </v-row>

          <div style="padding-bottom: 5rem"></div>
          <v-navigation-drawer
            v-model="comment"
            width="30%"
            height="100%"
            style="max-height: 100%"
            bottom
            fixed
            temporary
          >
            <template v-slot:prepend>
              <v-list-item two-line class="ml-16">
                <v-list-item-content>
                  <v-icon x-large>mdi-message-reply-outline</v-icon>
                </v-list-item-content>

                <v-list-item-content>
                  <v-list-item-title>Comments</v-list-item-title>
                  <!-- <v-list-item-subtitle>Logged In</v-list-item-subtitle> -->
                </v-list-item-content>
                <v-list-item-content>
                  <v-icon
                    v-if="$vuetify.breakpoint.mobile"
                    @click="comment = false"
                    x-large
                    >mdi-chevron-triple-down</v-icon
                  >
                  <!-- <v-list-item-subtitle>Logged In</v-list-item-subtitle> -->
                </v-list-item-content>
              </v-list-item>
              <v-divider></v-divider>
              <v-list-item v-if="isUser" class="ma-4"
                ><v-textarea
                  :loading="commentPosting"
                  auto-grow
                  placeholder="Write what you think about"
                  flat
                  reverse
                  rows="1"
                  v-model="commentmsg"
                  @click="writeCommentShow = true"
                >
                </v-textarea>
                <v-expand-transition>
                  <div v-if="writeCommentShow">
                    <v-icon @click="postComment">mdi-send</v-icon>
                  </div>
                </v-expand-transition>
              </v-list-item>
            </template>
            <v-skeleton-loader v-if="commentLoading" type="card@3" />
            <v-list v-else v-for="(c, i) in cs" :key="i" nav dense>
              <div>
                <v-list-item two-line>
                  <v-list-item-avatar class="clickable">
                    <avatar :size="30" :user="c.authorProfile"></avatar>
                  </v-list-item-avatar>

                  <v-list-item-content>
                    <v-list-item-title>{{
                      c.authorProfile.name
                    }}</v-list-item-title>
                    <time-ago
                      :refresh="60"
                      :datetime="c.entity_info.updateTime"
                      tooltip
                      long
                    ></time-ago>
                  </v-list-item-content>
                </v-list-item>
                <v-list-item>
                  <v-list-item-content>{{ c.content }} </v-list-item-content>
                </v-list-item>
                <v-list-item>
                  <v-spacer />
                  <v-icon @click="loadSub(c)">mdi-message-reply-outline</v-icon
                  >{{ c.subs.length }}
                  <v-list-item-action
                    ><v-icon @click="c.edit = !c.edit"
                      >mdi-reply</v-icon
                    ></v-list-item-action
                  >
                </v-list-item>
                <v-expand-transition>
                  <v-list-item v-if="c.edit" class="ma-4"
                    ><v-textarea
                      :loading="commentPosting"
                      auto-grow
                      placeholder="Write what you think about"
                      flat
                      reverse
                      rows="1"
                      v-model="c.tempC"
                    >
                    </v-textarea>
                    <div>
                      <v-icon @click="postSubComment(c)">mdi-send</v-icon>
                    </div>
                  </v-list-item>
                </v-expand-transition>
                <v-divider />
                <div v-if="c.showSub">
                  <v-list
                    class="ml-16"
                    v-for="(s, i) in c.subs"
                    :key="i"
                    nav
                    dense
                  >
                    <div>
                      <v-list-item two-line>
                        <v-list-item-avatar class="clickable">
                          <avatar :size="30" :user="s.authorProfile"></avatar>
                        </v-list-item-avatar>

                        <v-list-item-content>
                          <v-list-item-title>{{
                            s.authorProfile.name
                          }}</v-list-item-title>
                          <time-ago
                            :refresh="60"
                            :datetime="s.entity_info.updateTime"
                            tooltip
                            long
                          ></time-ago>
                        </v-list-item-content>
                      </v-list-item>
                      <v-list-item>
                        <v-list-item-content
                          >{{ s.content }}
                        </v-list-item-content>
                      </v-list-item>
                    </div>
                    <v-divider />
                  </v-list>
                </div>
              </div>
            </v-list>
            <v-skeleton-loader v-if="commentLoadingMore" type="card@3" />
            <v-list v-if="!nomore">
              <v-row justify="center" class="text-center">
                <v-btn
                  @click="loadMoreComments"
                  class="ma-5"
                  fab
                  dark
                  color="primary"
                >
                  <v-icon dark> mdi-plus </v-icon>
                </v-btn></v-row
              ></v-list
            >
            <v-list v-if="nomore && commentNum === 0">
              <v-list-item>
                <h1 class="ml-7">暂时没有评论</h1>
              </v-list-item>
            </v-list>
          </v-navigation-drawer>
        </div>
      </v-col>
    </v-row>
  </v-container>
</template>

<script lang="ts">
import Vue from "vue";
import Component from "vue-class-component";
import Editor from "../components/MO2Editor.vue";
import {
  DeleteArticle,
  GetArticle,
  GetCommentNum,
  GetComments,
  GetErrorMsg,
  GetUserData,
  GetUserDatas,
  globaldic,
  ShareToQQ,
  UpsertComment,
  UpsertSubComment,
  UserRole,
} from "@/utils";
import hljs from "highlight.js";
import { Blog, User, Comment, UserListData } from "@/models";
import Avatar from "@/components/UserAvatar.vue";
import { Prop } from "vue-property-decorator";
import { TimeAgo } from "vue2-timeago";
import DeleteConfirm from "@/components/DeleteConfirm.vue";
@Component({
  components: {
    Editor,
    Avatar,
    DeleteConfirm,
    TimeAgo,
  },
})
export default class ReadArticle extends Vue {
  @Prop()
  user: User;
  title = "";
  html = "";
  writeCommentShow = false;
  attrs = {
    class: "mb-6 mt-6",
    boilerplate: false,
    elevation: 0,
  };
  loading = true;
  blog: Blog;
  author: User;
  authorLoad = false;
  showDelete = false;
  draft = false;
  comment = false;
  commentmsg = "";
  p = 0;
  ps = 5;
  cs: Comment[] = [];
  commentLoading = true;
  commentPosting = false;
  commentLoadingMore = true;
  nomore = false;
  commentNum = 0;
  get isUser() {
    return this.user.roles && this.user.roles.indexOf(UserRole) >= 0;
  }
  get deleteContent() {
    return '你确定要删除"' + this.title + '"吗？';
  }
  share() {
    ShareToQQ({
      title: this.blog.title,
      pic: this.blog.cover,
      summary: this.blog.description,
      desc: this.blog.description,
    });
  }
  created() {
    if (this.$route.query["draft"]) {
      this.draft = (this.$route.query["draft"] as string) === "true";
    }
    GetArticle({ id: this.$route.params["id"], draft: this.draft })
      .then((val) => {
        this.loading = false;
        this.blog = val;
        this.title = val.title;
        this.html = val.content;
        GetUserData(this.blog.authorId).then((u) => {
          this.author = u;
          this.authorLoad = true;
        });
        GetCommentNum(this.blog.id).then((c) => {
          this.commentNum = c.count;
        });
        setTimeout(() => {
          // first, find all the code blocks
          document.querySelectorAll("code").forEach((block) => {
            // then highlight each
            hljs.highlightBlock(block);
          });
        }, 50);
      })
      .catch((err) => GetErrorMsg(err));
  }
  async loadSub(c: Comment) {
    if (c.showSub === true) {
      c.showSub = false;
      return;
    }
    if (c.subs.length > 0 && c.subs[0].authorProfile) {
      c.showSub = true;
      return;
    }
    const map: { [key: string]: UserListData } = {};
    (await GetUserDatas(c.subs.map((v) => v.aurhor))).forEach(
      (v) => (map[v.id] = v)
    );
    c.subs.forEach((v) => {
      v.authorProfile = map[v.aurhor];
      v.edit = false;
    });
    c.showSub = true;
  }
  async postSubComment(c: Comment) {
    this.commentPosting = true;
    const sub = await UpsertSubComment(c.id, { content: c.tempC });
    sub.authorProfile = this.user;
    c.showSub = true;
    c.subs.unshift(sub);
    c.tempC = "";
    this.commentPosting = false;
  }
  async postComment() {
    this.commentPosting = true;
    const c = await UpsertComment({
      article: this.blog.id,
      content: this.commentmsg,
    });
    c.authorProfile = this.user;
    c.edit = false;
    c.tempC = "";
    c.showSub = false;
    this.cs.unshift(c);
    this.commentmsg = "";
    this.commentNum++;
    this.commentPosting = false;
  }
  edit() {
    globaldic.article = `<h1>${this.title}</h1>${this.html}`;
    // UpsertBlog({ draft: true }, this.blog);
    this.$router.push("/edit/" + this.blog.id);
  }
  async loadComment() {
    this.comment = true;
    await this.loadMoreComments();
    this.commentLoading = false;
  }
  async loadMoreComments() {
    this.commentLoadingMore = true;
    const newCs = await GetComments(this.blog.id, {
      page: this.p++,
      pagesize: this.ps,
    });
    if (newCs.length < this.ps) {
      this.nomore = true;
    }
    const map: { [key: string]: UserListData } = {};
    (await GetUserDatas(newCs.map((v) => v.aurhor))).forEach(
      (v) => (map[v.id] = v)
    );
    newCs.forEach((v) => {
      v.authorProfile = map[v.aurhor];
      v.edit = false;
      v.tempC = "";
      v.showSub = false;
    });
    this.cs = this.cs.concat(newCs);
    this.commentLoadingMore = false;
  }
  deleteArticle() {
    this.showDelete = true;
  }
  confirmDelete() {
    DeleteArticle(this.blog.id, { draft: this.draft })
      .then(() => {
        this.$router.back();
      })
      .catch((err) => {
        alert(GetErrorMsg(err));
      });
  }
}
</script>
<style>
.v-skeleton-loader__text {
  border-radius: 6px;
  flex: 1 0 auto;
  height: 12px;
  margin-bottom: 6px !important;
}
</style>