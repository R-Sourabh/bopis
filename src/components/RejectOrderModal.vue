<template>
  <ion-header>
    <ion-toolbar>
      <ion-buttons slot="start">
        <ion-button @click="closeModal">
          <ion-icon slot="icon-only" :icon="closeOutline" />
        </ion-button>
      </ion-buttons>
      <ion-title>{{ translate("Reject Order") }}</ion-title>
    </ion-toolbar>
  </ion-header>
  <ion-content>
    <ion-item lines="none">
      <p>{{ translate('On rejecting this order, will be sent an email with alternate fulfilment options and this order will be removed from your dashboard.', { customerName: order?.customer?.name }) }}</p>
    </ion-item>
    <ion-list>
      <ion-list-header>{{ translate("Select reason") }}</ion-list-header>
      <ion-radio-group v-model="rejectReasonId">
        <ion-item v-for="reason in rejectReasons" :key="reason.enumId">
          <ion-radio :value="reason.enumId" label-placement="end" justify="start">{{ reason.description ? reason.description : reason.enumDescription ? reason.enumDescription : reason.enumId }}</ion-radio>
        </ion-item>
      </ion-radio-group>
    </ion-list>
    <ion-fab vertical="bottom" horizontal="end" slot="fixed">
      <ion-fab-button :disabled="!rejectReasonId" @click="confirmSave()">
        <ion-icon :icon="saveOutline" />
      </ion-fab-button>
    </ion-fab>
  </ion-content>
</template>
      
<script lang="ts">
import {
  IonButton,
  IonButtons,
  IonContent,
  IonFab,
  IonFabButton,
  IonHeader,
  IonIcon,
  IonItem,
  IonList,
  IonListHeader,
  IonTitle,
  IonToolbar,
  IonRadio,
  IonRadioGroup,
  alertController,
  modalController,
} from '@ionic/vue';
import { defineComponent } from 'vue';
import { closeOutline, saveOutline } from 'ionicons/icons';
import { mapGetters, useStore } from 'vuex'
import { translate } from '@hotwax/dxp-components';

export default defineComponent({
  name: "RejectOrderModal",
  components: {
    IonButton,
    IonButtons,
    IonContent,
    IonFab,
    IonFabButton,
    IonHeader,
    IonIcon,
    IonItem,
    IonList,
    IonListHeader,
    IonTitle,
    IonToolbar,
    IonRadio,
    IonRadioGroup
  },
  data() {
    return {
      rejectReasonId: ''
    }
  },
  computed: {
    ...mapGetters({
      order: "order/getCurrent",
      getProduct: 'product/getProduct',
      rejectReasons: 'util/getRejectReasons',
    })
  },
  methods: {
    closeModal() {
      modalController.dismiss({ dismissed: true });
    },
    async confirmSave() {
      const alert = await alertController
        .create({
          header: translate('Reject Order'),
          message: translate(`This order will be removed from your dashboard. This action cannot be undone.`, { space: '<br /><br />' }),
          buttons: [{
            text: translate('Cancel'),
            role: 'cancel'
          }, {
            text: translate('Reject'),
            handler: () => {
              const part = { ...this.order.part, items: this.order.part.items.map((item: any) => ({ ...item, reason: this.rejectReasonId })) };
              this.store.dispatch('order/setUnfillableOrderOrItem', { orderId: this.order.orderId, part }).then((resp) => {
                if (resp) {
                  // Mark current order as rejected
                  const order = { ...this.order, part: { ...this.order.part, items: [] }, rejected: true };

                  this.store.dispatch('order/updateCurrent', { order })
                }
                this.closeModal();
              })
            },
          }]
        });
      return alert.present();
    }
  },
  setup() {
    const store = useStore();

    return {
      closeOutline,
      saveOutline,
      store,
      translate
    };
  }
});
</script>