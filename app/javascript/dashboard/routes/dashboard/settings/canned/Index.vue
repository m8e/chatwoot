<template>
  <div class="column content-box">
    <button
      class="button nice icon success btn-fixed-right-top"
      @click="openAddPopup()"
    >
      <i class="icon ion-android-add-circle"></i>
      {{ $t('CANNED_MGMT.HEADER_BTN_TXT') }}
    </button>
    <!-- List Canned Response -->
    <div class="row">
      <div class="small-8 columns">
        <p
          v-if="!uiFlags.fetchingList && !records.length"
          class="no-items-error-message"
        >
          {{ $t('CANNED_MGMT.LIST.404') }}
        </p>
        <woot-loading-state
          v-if="uiFlags.fetchingList"
          :message="$t('CANNED_MGMT.LOADING')"
        />

        <table
          v-if="!uiFlags.fetchingList && records.length"
          class="woot-table"
        >
          <thead>
            <!-- Header -->
            <th
              v-for="thHeader in $t('CANNED_MGMT.LIST.TABLE_HEADER')"
              :key="thHeader"
            >
              {{ thHeader }}
            </th>
          </thead>
          <tbody>
            <tr
              v-for="(cannedItem, index) in records"
              :key="cannedItem.short_code"
            >
              <!-- Short Code  -->
              <td>{{ cannedItem.short_code }}</td>
              <!-- Content -->
              <td>{{ cannedItem.content }}</td>
              <!-- Action Buttons -->
              <td class="button-wrapper">
                <div @click="openEditPopup(cannedItem)">
                  <woot-submit-button
                    :button-text="$t('CANNED_MGMT.EDIT.BUTTON_TEXT')"
                    icon-class="ion-edit"
                    button-class="link hollow grey-btn"
                  />
                </div>
                <div @click="openDeletePopup(cannedItem, index)">
                  <woot-submit-button
                    :button-text="$t('CANNED_MGMT.DELETE.BUTTON_TEXT')"
                    :loading="loading[cannedItem.id]"
                    icon-class="ion-close-circled"
                    button-class="link hollow grey-btn"
                  />
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <div class="small-4 columns">
        <span v-html="$t('CANNED_MGMT.SIDEBAR_TXT')"></span>
      </div>
    </div>
    <!-- Add Agent -->
    <woot-modal :show.sync="showAddPopup" :on-close="hideAddPopup">
      <add-canned :on-close="hideAddPopup" />
    </woot-modal>

    <!-- Edit Canned Response -->
    <woot-modal :show.sync="showEditPopup" :on-close="hideEditPopup">
      <edit-canned
        v-if="showEditPopup"
        :id="selectedResponse.id"
        :edshort-code="selectedResponse.short_code"
        :edcontent="selectedResponse.content"
        :on-close="hideEditPopup"
      />
    </woot-modal>

    <!-- Delete Canned Response -->
    <delete-canned
      :show.sync="showDeleteConfirmationPopup"
      :on-close="closeDeletePopup"
      :on-confirm="confirmDeletion"
      :title="$t('CANNED_MGMT.DELETE.CONFIRM.TITLE')"
      :message="deleteMessage"
      :confirm-text="deleteConfirmText"
      :reject-text="deleteRejectText"
    />
  </div>
</template>
<script>
/* global bus */
import { mapGetters } from 'vuex';

import AddCanned from './AddCanned';
import EditCanned from './EditCanned';
import DeleteCanned from './DeleteCanned';

export default {
  components: {
    AddCanned,
    EditCanned,
    DeleteCanned,
  },
  data() {
    return {
      loading: {},
      showAddPopup: false,
      showEditPopup: false,
      showDeleteConfirmationPopup: false,
      selectedResponse: {},
      cannedResponseAPI: {
        message: '',
      },
    };
  },
  computed: {
    ...mapGetters({
      records: 'getCannedResponses',
      uiFlags: 'getUIFlags',
    }),
    // Delete Modal
    deleteConfirmText() {
      return `${this.$t('CANNED_MGMT.DELETE.CONFIRM.YES')} ${
        this.selectedResponse.short_code
      }`;
    },
    deleteRejectText() {
      return `${this.$t('CANNED_MGMT.DELETE.CONFIRM.NO')} ${
        this.selectedResponse.short_code
      }`;
    },
    deleteMessage() {
      return `${this.$t('CANNED_MGMT.DELETE.CONFIRM.MESSAGE')} ${
        this.selectedResponse.short_code
      } ?`;
    },
  },
  mounted() {
    // Fetch API Call
    this.$store.dispatch('getCannedResponse');
  },
  methods: {
    showAlert(message) {
      // Reset loading, current selected agent
      this.loading[this.selectedResponse.id] = false;
      this.selectedResponse = {};
      // Show message
      this.cannedResponseAPI.message = message;
      bus.$emit('newToastMessage', message);
    },
    // Edit Function
    openAddPopup() {
      this.showAddPopup = true;
    },
    hideAddPopup() {
      this.showAddPopup = false;
    },

    // Edit Modal Functions
    openEditPopup(response) {
      this.showEditPopup = true;
      this.selectedResponse = response;
    },
    hideEditPopup() {
      this.showEditPopup = false;
    },

    // Delete Modal Functions
    openDeletePopup(response) {
      this.showDeleteConfirmationPopup = true;
      this.selectedResponse = response;
    },
    closeDeletePopup() {
      this.showDeleteConfirmationPopup = false;
    },
    // Set loading and call Delete API
    confirmDeletion() {
      this.loading[this.selectedResponse.id] = true;
      this.closeDeletePopup();
      this.deleteCannedResponse(this.selectedResponse.id);
    },
    deleteCannedResponse(id) {
      this.$store
        .dispatch('deleteCannedResponse', id)
        .then(() => {
          this.showAlert(this.$t('CANNED_MGMT.DELETE.API.SUCCESS_MESSAGE'));
        })
        .catch(() => {
          this.showAlert(this.$t('CANNED_MGMT.DELETE.API.ERROR_MESSAGE'));
        });
    },
  },
};
</script>
