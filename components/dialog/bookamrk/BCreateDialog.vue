<template>
  <base-dialog
    :dialog="create_state"
    @close="close"
    :title="$vuetify.lang.t(`$vuetify.bookmark.create.header`)"
    icon="mdi-bookmark-plus-outline"
  >
    <div class="px-5">
      <p class="text-subtitle-2 text--secondary my-6">
        Lorem ipsum dolor, sit amet consectetur adipisicing elit. Cum
        reprehenderit quis deserunt aliquam eius vero aperiam velit itaque
      </p>
      <!-- the info text : about this dialog -->
      <validation-observer ref="url">
        <validation-provider
          v-slot="{ errors }"
          rules="required|url"
          :name="$vuetify.lang.t('$vuetify.bookmark.form.link')"
        >
          <v-text-field
            :label="$vuetify.lang.t('$vuetify.bookmark.form.link')"
            v-model="url"
            prepend-icon="mdi-link"
            class="text-capitalize"
            :error-messages="errors"
            name="url"
            required
          ></v-text-field>
          <!-- input element  -->
        </validation-provider>
        <!-- validat for url : required and must be url  -->
      </validation-observer>
      <!-- validation observer -->
    </div>
    <template v-slot:actions>
      <v-btn :loading="loading" color="primary" class="mb-4" @click="save_url">
        <v-icon size="20px" class="px-1">mdi-content-save-outline </v-icon>
        {{ $vuetify.lang.t(`$vuetify.bookmark.create.save_btn`) }}
      </v-btn>
    </template>
    <!-- card action -->
  </base-dialog>
  <!-- card body for dialog  -->
</template>

<script>
import { mapGetters, mapMutations, mapActions } from 'vuex'
import BaseDialog from '@/components/dialog/BaseDialog'
export default {
  name: 'create-bookmark-dialog',

  components: {
    BaseDialog,
  },
  // --------- componets --------

  data: () => ({
    loading: false,
    url: '',
  }),
  // --------- data ------------

  methods: {
    ...mapActions({
      b_current: 'bookmark/current_bookmark',
    }),
    // vuex - actions
    ...mapMutations({
      create_state_toggle: 'bookmark/create_state_toggle',
      edit_state_toggle: 'bookmark/edit_state_toggle',
      set_message: 'set_message',
      toggle_message: 'toggle_message',
    }),
    // vuex - mutations
    /**
     * for close and rest dialog
     *
     */
    close() {
      this.create_state_toggle()
      this.url = ''
      this.$refs.url.reset()
    },
    // ------------ close -----------------

    /**
     * check if input is valid after that send data to database and rest dialog
     *
     */
    save_url() {
      this.$refs.url.validate().then(async (success) => {
        if (success) {
          this.loading = true
          let data = {
            url: this.url,
            folder_id: this.get_folder.id,
            title: 'no data',
          }
          await this.$axios
            .post('bookmarks/', data, {
              auth: {
                username: 'root',
                password: 'root',
              },
            })
            .then((res) => res.data)
            .then((data) => {
              this.b_current(data.id)
              this.loading = false
              this.set_message({
                type: 'success',
                text: 'Added bookmark successful',
              })
              this.close()
              setTimeout(() => {
                this.edit_state_toggle()
              }, 300)
            })
            .catch((err) => {
              this.loading = false
              this.set_message({
                type: 'error',
                text: 'Added bookmark error',
              })
            })
          this.toggle_message(true)
        }
      })
    },
    // send data to server then open update dialog
    // ------------ save_url -------------
    async send_data() {},
  },
  // ------------- methods ---------------

  computed: {
    ...mapGetters({
      create_state: 'bookmark/create_state',
      get_folder: 'folder/get_folder',
    }),
    //vuex - Getters
    t_dialog() {
      return this.$store.state.b_create_dialog
    },
  },
}
</script>

