<template>
  <div>
    <div class="ui stackable grid container" v-if="profile">
      <div class="four wide column">
        <b-image :src="profile.s3_id" ratio="1x1" rounded />
        <h3>{{ `Welcome to your garden, @${profile.username}.` }}</h3>
      </div>
      <div class="eight wide column">
        <div class="ui segment">
          <h3 class="ui medium dividing header">
            {{ `${profile.username}: ${profile.status}` }}
          </h3>
          <div class="status-form" v-if="user.id === profile.id">
            <b-field label="Update your Status">
              <b-input
                name="newStatus"
                type="textarea"
                v-model="newStatus"
                placeholder="What are you up to?"
              />
            </b-field>
            <b-button type="is-primary" @click="updateStatus">
              Update
            </b-button>
          </div>
          <div v-else>
            <b-switch v-model="friend" type="is-info">{{
              friend ? "Following" : "Not Following"
            }}</b-switch>
          </div>
        </div>
      </div>
      <div class="gardens" v-if="gardens.length">
        <Carousel :gardens="gardens" />
      </div>
    </div>
  </div>
</template>

<script>
import Carousel from "./garden-carousel.vue";
import axios from "axios";

export default {
  name: "UserProfile",
  components: {
    Carousel,
  },
  props: ["user"],
  data() {
    return {
      gardens: [],
      newStatus: "",
      profile: {},
      friend: false,
    };
  },
  watch: {
    friend: async function () {
      const action = !this.friend ? "remove" : "add";
      const method = !this.friend ? "delete" : "post";
      try {
        const response = await axios({
          method,
          url: `/friend/${action}`,
          data: {
            id_user: this.user.id,
            id_friend: this.profile.id,
          },
        });
      } catch (err) {
        this.$buefy.toast.open({
          duration: 1000,
          message: "Couldn't update Friendship",
          type: "is-danger",
        });
        this.$log.error("Error Changing Friend ", err);
      }
    },
  },
  methods: {
    async updateStatus() {
      try {
        const response = await axios({
          method: "put",
          url: "/user/userupdate",
          data: {
            id: this.user.id,
            info: { status: this.newStatus },
          },
        });
        this.profile.status = this.newStatus;
        this.newStatus = "";
      } catch (err) {
        this.$buefy.toast.open({
          duration: 1000,
          message: "Couldn't update Status",
          type: "is-danger",
        });
        this.$log.error("Error Changing Status ", err);
      }
    },
  },
  async mounted() {
    if (this.$route.query.id) {
      try {
        const response = await axios({
          method: "get",
          url: "/user/getuser",
          params: {
            id: this.$route.query.id,
          },
        });
        const friendResponse = await axios({
          method: "get",
          url: "/friend/status",
          params: {
            userId: this.user.id,
            friendId: this.$route.query.id,
          },
        });
        this.profile = response.data;
        this.friend = friendResponse.data;
      } catch (err) {
        this.$log.error(err);
        this.$buefy.toast.open({
          duration: 1000,
          message: "User Profile Not Found",
          type: "is-danger",
        });
        return;
      }
    } else {
      this.profile = this.user;
    }
    try {
      const response = await axios({
        method: "get",
        url: "/garden/user",
        params: {
          id: this.profile.id,
        },
      });
      this.gardens = response.data;
    } catch (err) {
      this.$log.error(err);
      this.$buefy.toast.open({
        duration: 1000,
        message: "Gardens Not Found",
        type: "is-danger",
      });
      return;
    }
  },
};
</script>
