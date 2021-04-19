// eslint-disable-next-line vue/valid-template-root
<template>
  <div class="container-fluid">
    <div class="row justify-content-center">
      <div class="col-lg-9">
        <div class="col-12">
          <input type="text" :class="{ invalid: error }" v-model="username" />
          <p v-if="error && username">{{ error }}</p>
        </div>
      </div>
    </div>
    <div class="row">
      <div class="col-lg-4 col-12 d-inline-block">
        <b-form-select
          id="reposOption"
          v-model="repoSelected"
          v-if="!enabledAuto"
          :disabled="!loadingRepos"
          size=" sm"
          class="col-8 mt-3"
        >
          <!-- eslint-disable-next-line vue/no-unused-vars -->
          <b-form-select-option
            v-for="(repo, index) in processedRepos"
            :key="index"
            :value="repo.full_name"
            >{{ repo.name }} ({{ repo.size }})
          </b-form-select-option>
        </b-form-select>
        <input
          v-else
          type="text"
          v-model="searched"
          class="inputSelectBox col-8 mt-3 custom-select custom-select- sm"
        />
        <b-form-select
          v-model="sortSelect"
          :options="sortOptions"
          class="col-2 offset-1 mt-3"
          size="col-8 sm"
        ></b-form-select>
        <div class="mt-3">
          Selected: <strong>{{ repoSelected }}</strong>
          <input
            type="checkbox"
            id="scales"
            name="scales"
            v-model="enabledAuto"
            :checked="enabledAuto"
          />

          <label for="scales">Autocomplet</label>
          <div
            class="autocomplet-modal"
            v-if="enabledAuto && processedRepos && searched && suggestionOpened"
          >
            <ul>
              <li
                class="li-modal"
                v-for="(item, i) in processedRepos"
                :key="i"
                @click="
                  repoSelected = item.full_name;
                  suggestionOpened = false;
                "
              >
                {{ item.name }}
              </li>
            </ul>
          </div>
        </div>
      </div>
      <div class="col-md-8 col-12 py-4" :class="{ spinner: commitLoading }">
        <b-col md="6" class="mb-3" v-if="commitLoading">
          <b-icon
            icon="arrow-clockwise"
            animation="spin"
            font-scale="4"
          ></b-icon>
        </b-col>
        <ul class="commit-list">
          <li
            class="TimelineItem"
            v-for="(commit, index) in processedCommit"
            :key="index"
          >
            <b-icon
              class="octicon"
              icon="arrow-return-right"
              font-scale="1"
            ></b-icon>
            <h4 class="commit-date">Commits on {{ commit.date }}</h4>
            <div
              class="commit-box my-2"
              v-for="(group, j) in commit.groups"
              :key="j"
            >
              <div class="commit-header">
                <div class="author-avatar m-auto">
                  <img :src="group.author.avatar_url" alt="" class="avatar" />
                </div>
                <div class="author-name m-auto">
                  <p class="h2 name">{{ group.commit.author.name }}</p>
                </div>
              </div>
              <div class="commit-body">
                <p>{{ group.commit.message }}</p>
              </div>
            </div>
          </li>
        </ul>
        <ul
          class="commit-list justify-content-center"
          v-if="!commitLoading && repoSelected && processedCommit.length == 0"
        >
          <li>
            <h2>
              No Availble Commits for the
              <strong>{{ repoSelected }}</strong> repo
            </h2>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
// import axios from 'vue-axios'

export default {
  name: "GithubMain",
  data() {
    return {
      fullRepoName: "",
      //   username: "TheGlitch1",
      username: this.$route.params.username,
      password: "",
      token: "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",
      repos: [],
      repo_names: [],
      error: "",
      repoSelected: "",
      commits: [],
      sortOptions: ["Name", "Size"],
      sortSelect: "",

      enabledAuto: false,
      searched: "",
      commitLoading: false,
      loadingRepos: false,
      watch: "",
      suggestionOpened: false,
    };
  },
  created() {
    this.getRepo();
  },
  methods: {
    async getRepo() {
      this.error = "";
      // eslint-disable-next-line no-unused-vars
      let config = {
        headers: {
          //   Authorization: "token" + btoa(this.username + ":" + this.password),
          Authorization: "Token" + this.token,
        },
      };

      // eslint-disable-next-line no-unused-vars
      let data = {
        HTTP_CONTENT_LANGUAGE: self.language,
      };
      // eslint-disable-next-line no-unused-vars
      let api =
        "https://api.github.com/search/repositories?q=user:" + this.username;
      // eslint-disable-next-line no-unused-vars
      let publicApi =
        "https://api.github.com/users/" + this.username + "/repos?";
      await this.axios
        .get(api, data, config)
        .then((response) => {
          this.repos = response.data.items;
          this.loadingRepos = true;
        })
        .catch((error) => {
          console.log(error);
          this.error = "username not valide";
          if (error.status == "422") {
            // this.error = ;
          }
        });
    },
    async getCommit() {
      // eslint-disable-next-line no-unused-vars
      let config = {
        headers: {
          Accept: "application/vnd.github.cloak-preview",
        },
      };

      let api =
        "https://api.github.com/search/commits?q=repo:" +
        this.repoSelected +
        " author-date:2018-12-30..2021-04-04";

      await this.axios
        .get(api, config)
        .then((response) => {
          this.commits = response.data.items;
          console.log("showing commits-items");
          console.log(response.data.items);

          //   console.log(response.data.items)
          this.commitLoading = false;
        })
        .catch((error) => {
          console.log(error);
        });
    },
    openOption() {
      attemptFocus(this.$refs.input);
      // $("#reposOption").openSelect();
      // document.getElementById("reposOption").openSelect();
    },
  },
  watch: {
    // whenever question changes, this function will run
    repoSelected: function () {
      this.commitLoading = true;
      this.getCommit();
      this.watch = this.repoSelected;
    },
    username: function () {
      this.loadingRepos = true;
      this.repos = [];
      this.getRepo();
    },
  },
  filters: {
    min(value) {
      return value.substr(0, 10);
    },
  },
  computed: {
    processedRepos: function () {
      let repos = this.repos;
      //autocomplet
      if (this.enabledAuto && this.searched !== "" && this.searched !== null) {
        this.suggestionOpened = true;
        repos = this.repos.filter((item) => {
          return item.name
            .toLowerCase()
            .startsWith(this.searched.toLowerCase());
        });
      }
      //sorting
      if (repos) {
        if (this.sortSelect == "Name") {
          repos.sort((a, b) => a.name.localeCompare(b.name));
        }
        if (this.sortSelect == "Size") {
          repos.sort(function (a, b) {
            return b.size - a.size;
          });
        }
      }
      return repos;
    },
    processedCommit: function () {
      let commits;

      commits = this.commits.reduce((commits, commit) => {
        const date = commit.commit.author.date.split("T")[0];
        //   !commits[date] ? commits[date] = [] :
        if (!commits[date]) {
          commits[date] = [];
        }
        commits[date].push(commit);
        return commits;
      }, {});
      console.log(commits);
      commits = Object.keys(commits).map((date) => {
        return {
          date,
          groups: commits[date],
        };
      });
      // eslint-disable-next-line no-unused-vars
      //   const groupArrays = Object.keys(commits).map((date) => {
      //     return {
      //         date,
      //         commit: commits[date]
      //     };
      //     });

      return commits;
    },
  },
};
</script>
<style scoped>
.row {
  margin: 0 !important;
}
.octicon {
  top: 42px;
  padding-top: 2px;
  position: absolute;
}
.TimelineItem {
  position: relative;
  margin-bottom: 10px;
  padding: 16px 0;
  margin-left: 16px;
}
.TimelineItem:before {
  position: absolute;
  top: 58px;
  bottom: 0;
  left: 5px;
  display: block;
  width: 2px;

  content: "";
  background-color: #e2e0e0;
}
ul {
  list-style: none;
  text-align: left;
}

.commit-box {
  margin-left: 100px;
  border-width: 0.5px;
  border-style: solid;
  border-color: azure;
  border-bottom-right-radius: 6px;
  border-bottom-left-radius: 6px;
  background-color: #e2e0e0;
  border-top-right-radius: 6px;
  border-top-left-radius: 6px;
}

.commit-header {
  background-color: #f6f8fa;
  /* border-bottom: 1px solid #f6f8fa; */
  height: 65px;
}

.commit-body {
  padding: 4px 16px;
}

.author-avatar {
  width: 60px;
  height: 60px;
  float: left;
  padding: 9px;
}

.author-avatar img {
  width: 100%;
}

.author-name {
  padding: 5px;
}
.autocomplet-modal {
  background-color: #ffffff;
  position: absolute;
  top: 53px;
  left: 40px;
  width: 64%;
}
li.li-modal:hover {
  background-color: #e1dede;
}
</style>