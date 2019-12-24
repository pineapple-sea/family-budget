<template>
  <div class="main">
    <h1 class="main-title">Сімейний бюджет</h1>
    <v-stepper v-model="stepperNum">
      <v-stepper-header>
        <v-stepper-step editable step="1" color="orange">Члени сім'ї</v-stepper-step>

        <v-divider></v-divider>

        <v-stepper-step editable step="2" color="orange">Витрати</v-stepper-step>

        <v-divider></v-divider>

        <v-stepper-step editable step="3" color="orange">Збереження</v-stepper-step>
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
                Члени сім'ї
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
                  Додати члена сім'ї
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
            @click="stepperNum = 2"
          >
            Продовжити
          </v-btn>
        </v-stepper-content>

        <v-stepper-content step="2">
          <div class="cards-wrapper">

            <v-card
              class="card add-card"
            >
              <v-card-title
                  class="card-title"
              >
                Сімейні витрати
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
                  Додати витрати
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
            @click="stepperNum = 3"
          >
            Продовжити
          </v-btn>

          <v-btn
            color="orange"
            @click="stepperNum = 1"
          >
            Повернутися
          </v-btn>
        </v-stepper-content>


        <v-stepper-content step="3">
          <div class="cards-wrapper">

            <v-card
              class="card add-card"
            >
              <v-card-title
                  class="card-title"
              >
                Сімейні збереження
              </v-card-title>
              <v-img
                class="add-card__img"
                width="200px"
                src="https://pngimg.com/uploads/piggy_bank/piggy_bank_PNG104.png"
              >
              </v-img>
                <p class="card__text">
                  Збереження на такий період років: {{years}} становить : {{getTotalSaved}}
                </p>
              <div class="card__saved">
                <v-btn @click="minusYear" class="mx-3" fab dark small color="orange">
                  <v-icon dark>mdi-minus</v-icon>
                </v-btn>
                <v-btn @click="plusYear" class="mx-3" fab dark small color="orange">
                  <v-icon dark>mdi-plus</v-icon>
                </v-btn>
              </div>
            </v-card>
          </div>
          <v-btn
            color="orange"
            @click="stepperNum = 2"
          >
            Повернутися
          </v-btn>
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
      stepperNum: 1,
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
  created() {
    this.stepperNum = localStorage.getItem('stepperNum') || 1;
  },
  mounted() {
    let openRequest = indexedDB.open("store", 1);

    openRequest.onupgradeneeded = function() {
      let db = openRequest.result;

      if (!db.objectStoreNames.contains('members')) {
        db.createObjectStore('members', {keyPath: 'id', autoIncrement: true});
      };
      if (!db.objectStoreNames.contains('outlays')) {
        db.createObjectStore('outlays', {keyPath: 'id', autoIncrement: true});
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
      const newMemberId = (this.members[this.members.length - 1] || {}).id +1 || 0;
      const newMember = {...member, id: newMemberId};
      this.members.push(newMember);

      let transaction = this.db.transaction("members", "readwrite");
      let members = transaction.objectStore("members");
      let request = members.add(newMember);
    },
    editMember(editedMember) {
      this.members.forEach((member, i) => {
        if (editedMember.id === member.id) {
          this.members.splice(i, 1, editedMember)
        }
      });

      let transaction = this.db.transaction("members", "readwrite");
      let members = transaction.objectStore("members");
      let request = members.put(editedMember);
    },
    deleteMember(memberId) {
      this.members.forEach((member, i) => {
        if (memberId === member.id) {
          this.members.splice(i, 1)
        }
      });

      let transaction = this.db.transaction("members", "readwrite");
      let members = transaction.objectStore("members");
      let request = members.delete(memberId);
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
        this.outlays.forEach((globalOutLay, i) => {
          if (globalOutLay.id === outlay.id) {
            this.outlays.splice(i, 1, outlay);
          }
        });

        let transaction = this.db.transaction("outlays", "readwrite");
        let outlaysStore = transaction.objectStore("outlays");
        let request = outlaysStore.put(outlay);
      });
    },
    deleteOutlays(outlays) {
      outlays.forEach(outlay => {
        this.outlays.forEach((globalOutLay, i) => {
          if (globalOutLay.id === outlay.id) {
            this.outlays.splice(i, 1);
          }
        });

        let transaction = this.db.transaction("outlays", "readwrite");
        let outlaysStore = transaction.objectStore("outlays");
        let request = outlaysStore.delete(outlay.id);
      });
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
  },
  watch: {
    stepperNum(newStepperNum) {
      localStorage.setItem('stepperNum', newStepperNum);
    }
  }
}
</script>
<style lang="scss">
  .card__saved {
    width: 100%;
    display: flex;
    flex-direction: row;
    justify-content: center;
    margin-bottom: 15px;
  }
  .card__text {
    color: #ffa500;
    font-size: 15pt;
    display: flex;
    flex-direction: row;
    justify-content: center;
    margin: 10px 0 10px 0;
  }
  .main-title {
    font-family: monospace;
    color: #ffa500;
    text-align: center;
    margin: 40px 0;
  }
  .card-title {
    color: #ffa500;
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
