<template>
  <div class="main">
    <h1 class="main-title">FAMILY BUDGET CONTROLLER</h1>
    <v-stepper v-model="e1">
      <v-stepper-header>
        <v-stepper-step editable :complete="e1 > 1" step="1" color="orange">Members</v-stepper-step>

        <v-divider></v-divider>

        <v-stepper-step editable :complete="e1 > 2" step="2" color="orange">Outlays</v-stepper-step>

        <v-divider></v-divider>

        <v-stepper-step editable step="3" color="orange">Saved</v-stepper-step>
      </v-stepper-header>

      <v-stepper-items>
        <v-stepper-content step="1">
          <div class="cards-wrapper">
            <v-card
              class="card add-card"
            >
              <v-card-title
                  class="card-title"
              >
                Family members
              </v-card-title>
              <v-img
                class="add-card__img"
                width="200px"
                src="https://image.flaticon.com/icons/png/512/1530/1530891.png"
              >
              </v-img>
              
              <v-card-actions>
                <v-btn @click="openAddMemberPopup" class="mx-auto" color="orange" text
                >
                  Add member
                </v-btn>
              </v-card-actions>
            </v-card>

            <div class="cards">
              <v-card 
                v-for="member in members"
                @click="openEditMemberPopup(member)"
                :key="member.userName"
                class="card"
                outlined
              >
                <v-card-title class="card-title">
                  {{member.firstName}} {{member.lastName}}
                </v-card-title>
                <v-card-text>
                  {{member.salary}}
                </v-card-text>
              </v-card>
            </div>
          </div>

          <v-btn
            color="orange"
            @click="e1 = 2"
          >
            Continue
          </v-btn>

          <v-btn text>Cancel</v-btn>
        </v-stepper-content>

        <v-stepper-content step="2">
          <div class="cards-wrapper">

            <v-card
              class="card add-card"
            >
              <v-card-title
                  class="card-title"
              >
                Family outlays
              </v-card-title>
              <v-img
                class="add-card__img"
                width="200px"
                src="http://krassmart.ru/images/companies/1/5f44f3160a09b51b4fa4634ecdff62dd-money-icon-by-vexels.png?1529992682589"
              >
              </v-img>

              <v-card-actions>
                <v-btn @click="openAddOutlayPopup" class="mx-auto" color="orange" text
                >
                  Add cost
                </v-btn>
              </v-card-actions>
            </v-card>

            <div class="cards">
              <v-card 
                v-for="(outlay, type) in outlaysObj"
                @click="openEditCostPopup(outlay)"
                :key="outlay.costName"
                class="outlay"
                max-width="344"
                outlined
              >
                <v-card-title class="card-title">
                  {{type}}
                </v-card-title>
                <v-card-text>
                </v-card-text>
              </v-card>
            </div>
          </div>

          <v-btn
            color="orange"
            @click="e1 = 3"
          >
            Continue
          </v-btn>

          <v-btn text>Cancel</v-btn>
        </v-stepper-content>

        <v-stepper-content step="3">
          <v-card
            height="200px"
          >
            <div class="total">
              <p>
                Saved for {{years}} year: {{getTotalSaved}}
              </p>
              <v-btn @click="minusYear" class="mx-2" fab dark small color="orange">
                <v-icon dark>mdi-minus</v-icon>
              </v-btn>
              <v-btn @click="plusYear" class="mx-2" fab dark small color="orange">
                <v-icon dark>mdi-plus</v-icon>
              </v-btn>
            </div>
          </v-card>

          <v-btn
            color="orange"
            @click="e1 = 1"
          >
            Continue
          </v-btn>

          <v-btn text>Cancel</v-btn>
        </v-stepper-content>
      </v-stepper-items>
    </v-stepper>

    <MemberDialog
      v-model="memberDialog"
      :mode="memberDialogMode"
      :member="editingMember"
      @addMember="addMember"
      @editMember="editMember"
      @deleteMember="deleteMember"
    />
    <OutlayDialog
      v-model="outlayDialog"
      :mode="outlayDialogMode"
      :outlays="editingOutlay"
      @addOutlay="addOutlay"
      @editOutlay="editOutlay"
      @deleteOutlays="deleteOutlays"
    />
  </div>
</template>

<script>
import MemberDialog from '~/components/MemberDialog.vue'
import OutlayDialog from '~/components/OutlayDialog.vue'

export default {
  components: {
    MemberDialog,
    OutlayDialog
  },
  data() {
    return {
      e1: 0,
      years: 1,
      db: null,

      members: [],
      memberDialog: false,
      memberDialogMode: '',
      editingMember: {},

      outlays: [],
      outlayDialog: false,
      outlayDialogMode: '',
      editingOutlay: {}
    }
  },
  mounted() {
    let openRequest = indexedDB.open("store", 1);

    openRequest.onupgradeneeded = function() {
      let db = openRequest.result;

      if (!db.objectStoreNames.contains('members')) {
        db.createObjectStore('members', {keyPath: 'id'});
      };
      if (!db.objectStoreNames.contains('outlays')) {
        db.createObjectStore('outlays', {keyPath: 'id'});
      };
    };

    openRequest.onsuccess = () => {
      this.db = openRequest.result;

      let transactionMembers = this.db.transaction("members", "readonly");
      let members = transactionMembers.objectStore("members");
      members.getAll().onsuccess = (e) => {
        this.members = e.target.result;
      };

      let transactionOutlays = this.db.transaction("outlays", "readonly");
      let outlays = transactionOutlays.objectStore("outlays");
      outlays.getAll().onsuccess = (e) => {
        this.outlays = e.target.result;
      };
    };
  },
  methods: {
    openAddMemberPopup() {
      this.editingMember = {};
      this.memberDialogMode = 'add';
      this.memberDialog = !this.memberDialog;
    },
    openEditMemberPopup(member) {
      this.editingMember = member;
      this.memberDialogMode = 'edit';
      this.memberDialog = !this.memberDialog;
    },
    addMember(member) {
      const newMemberId = ((this.members[this.members.length - 1] || {}).id || -1) + 1;
      const newMember = {...member, id: newMemberId};
      this.members.push(newMember);

      let transaction = this.db.transaction("members", "readwrite");
      let members = transaction.objectStore("members");
      let request = members.add(newMember);
    },
    editMember(member) {
      this.members.splice(member.id, 1, member)

      let transaction = this.db.transaction("members", "readwrite");
      let members = transaction.objectStore("members");
      let request = members.put(member);
    },
    deleteMember(memberId) {
      this.members.splice(memberId, 1)

      let transaction = this.db.transaction("members", "readwrite");
      let members = transaction.objectStore("members");
      let request = members.delete(memberId);
    },
    minusYear() {
      if (this.years > 1) {
        this.years--;
      }
    },
    plusYear() {
      if (this.years <= 100) {
        this.years++;
      }
    },
    openAddOutlayPopup() {
      this.editingOutlay = {};
      this.outlayDialogMode = 'add';
      this.outlayDialog = !this.outlayDialog;
    },
    openEditCostPopup(outlay) {
      this.editingOutlay = outlay;
      this.outlayDialogMode = 'edit';
      this.outlayDialog = !this.outlayDialog;
    },
    addOutlay(outlay) {
      const newOutlayId = (this.outlays[this.outlays.length - 1] || {}).id + 1 || 0;
      const newOutlay = {...outlay, id: newOutlayId};
      this.outlays.push(newOutlay);

      let transaction = this.db.transaction("outlays", "readwrite");
      let outlays = transaction.objectStore("outlays");
      let request = outlays.add(newOutlay);
    },
    editOutlay(outlays) {
      outlays.forEach(outlay => {
        this.outlays.splice(outlay.id, 1, outlay);

        let transaction = this.db.transaction("outlays", "readwrite");
        let outlaysStore = transaction.objectStore("outlays");
        let request = outlaysStore.put(outlay);
      });
    },
    deleteOutlays(outlays) {
      outlays.forEach(outlay => {
        this.outlays.splice(outlay.id, 1);

        let transaction = this.db.transaction("outlays", "readwrite");
        let outlaysStore = transaction.objectStore("outlays");
        let request = outlaysStore.delete(outlay.id);

        request.onsuccess = (e) => {
          console.log(request)
          console.log(e);
        }
      });
    }
  },
  computed: {
    getTotalSallary() {
      const totalSalaries = this.members.reduce((sum, member) => {
        return sum += +member.salary
      }, 0)
      return totalSalaries * 12 * this.years;
    },
    getTotalOutlays() {
      const totalOutlays = this.outlays.reduce((sum, outlay) => {
        return sum += +outlay.cost
      }, 0);
      return totalOutlays * 12 * this.years;
    },
    getTotalSaved() {
      return this.getTotalSallary - this.getTotalOutlays;
    },
    outlaysObj() {
      return this.outlays.reduce((obj, outlay) => {
        if (obj[outlay.costType]) {
          obj[outlay.costType].push(outlay);
          return obj;
        }

          obj[outlay.costType] = [outlay];
          return obj;
      }, {});
    },
  }
}
</script>
<style lang="scss">
  .total {
    color: #cc99ff;
  }
  .main-title {
    font-family: monospace;
    color: #cc99ff;
    text-align: center;
    margin: 40px 0;
  }

  .cards-wrapper {
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    margin-bottom: 25px;

    .add-card {
      margin-bottom: 25px;

      .add-card__img {
        margin: 0 auto;
        background-size: contain;
      }
    }

    .cards {
      width: 100%;
      display: flex;
      flex-direction: row;
      justify-content: space-between;

      .card {
        width: 200px;
      }
    }
  }

</style>
