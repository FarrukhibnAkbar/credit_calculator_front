<template>
  <div>
    <div class="form-container">
      <div class="d-flex align-items-center justify-content-between">

        <div class="d-flex flex-column align-items-center justify-content-center">
          <!-- Kredit Summasi -->
          <div class="mt-1">
            <label class="basic_label" for="principal">Kredit Summasi:</label>
            <input
              class="basic_input"
              v-model="principal"
              type="text" 
              id="principal"
              placeholder="Kredit summasini tanlang"
            />
          </div>

          <!-- Yillik Foiz Stavkasi -->
          <div class="mt-1">
            <label class="basic_label" for="annual_rate">Yillik Foiz Stavkasi (%):</label>
            <input class="basic_input" v-model="annualRate" type="number" min="1" max="30" step="0.1" id="annual_rate" placeholder="Foizni tanlang"/>
          </div>
        </div>
        
        <div class="d-flex flex-column align-items-center justify-content-center">
          <!-- Muddat (Oylarda) -->
          <div class="mt-1">
            <label class="basic_label" for="months">Muddat (Oylarda):</label>
            <input class="basic_input" v-model="months" type="number" min="1" max="24" step="1" id="months" placeholder="Muddatni tanlang"/>
          </div>
        
          <!-- Kredit Turi -->
          <div class="mt-1">
            <label for="credit_type">Kredit Turi:</label>
            <select class="basic_select" v-model="creditType" id="credit_type">
              <option value="differential">Differential</option>
              <option value="annuitet">Annuitet</option>
            </select>
          </div>
        </div>
      </div>

      <!-- Hisoblash tugmasi -->
      <div class="d-flex mt-1">
        <button class="basic_select_btn" @click="calculateCredit">Hisoblash</button>
      </div>

      <!-- Xatoliklar -->
      <div v-if="errorMessage" class="error-message">
        <p>{{ errorMessage }}</p>
      </div>
    </div>

    <!-- Natija -->
    <div v-if="responseData && responseData.length" class="table-container">
      <table>
        <thead>
          <tr>
            <th>№</th>
            <th>Sana</th>
            <th>Foiz to'lovlari(ming so'm)</th>
            <th>Asosiy miqdor(ming so'm)</th>
            <th>Umumiy to'lov(ming so'm)</th>
            <th>Qoldiq(ming so'm)</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(item, index) in responseData" :key="index">
            <td>{{ item.number }}</td>
            <td>{{ item.date }}</td>
            <td>{{ item.interest }}</td>
            <td>{{ item.principal }}</td>
            <td>{{ item.total_payment }}</td>
            <td>{{ item.remaining_debt }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      principal: 1000000.0,
      annualRate: 10,
      months: 12,
      creditType: 'differential',
      responseData: [],
      errorMessage: ''
    };
  },

  methods: {
    async calculateCredit() {
      this.errorMessage = '';
      try {
        const response = await axios.post('http://194.233.90.79/api/v1/calculate', {
          principal: parseInt(this.principal),
          annual_rate: this.annualRate,
          months: this.months,
          credit_type: this.creditType
        });

        if (response.data.status === 'OK') {
          this.responseData = response.data.data[0];
        } else {
          this.errorMessage = 'Xato yuz berdi. Iltimos, qayta urinib ko\'ring.';
        }
      } catch (error) {
        this.errorMessage = 'API so\'rovi bajarilmadi: ' + error.message;
      }
    },

    formatPrincipal(event) {
      // Remove all non-numeric characters except for the space
      let value = event.target.value.replace(/\D/g, '');

      // Add a space every three digits from the end
      value = value.replace(/\B(?=(\d{3})+(?!\d))/g, ' ');

      // Update the principal value with the formatted number
      this.principal = value;
    }
  },
  watch: {
    annualRate(val) {
      if (val>=30) {
        alert("yillik foiz 30 foizdan oshmasin");
        this.annualRate=30;
      }
    },
    principal(val) {
      if (val>=100000000) {
        alert("sizga hozirda max 100 000 000 so'm kredit beraolamiz.");
        this.principal=100000000;
      }
    }
  }
};
</script>

<style scoped>
.d-flex {
  display: flex;
} 

.flex-column {
  flex-direction: column;
}

.align-items-center {
  align-items: center;
}

.justify-content-between {
  justify-content: space-between;
}

.justify-content-center {
  justify-content: center;
}

.mt-1 {
  margin-top: 1rem;
}

.form-container {
  display: flex;
  flex-direction: column;
  max-width: 800px;
  margin: 20px auto;
}

.basic_label {
  font-size: 14px;
  color: #0E121B;
  text-align: left;
}

.basic_input, .basic_select {
  color: #0E121B;
  background: #fff;
  border: 1px #E1E4EA solid;
  padding: 10px 12px;
  border-radius: 10px;
  font-size: 14px;
  margin-left: 8px;
}

.basic_select_btn {
  border-radius: 10px;
  border: 1px solid rgba(255, 255, 255, 0.12);
  background: linear-gradient(180deg, rgba(255, 255, 255, 0.16) 0%, rgba(255, 255, 255, 0.00) 100%), #335CFF;
  color: #fff;
  box-shadow: 0px 1px 2px 0px rgba(14, 18, 27, 0.24), 0px 0px 0px 1px #335CFF;
  padding: 10px;
}

.table-container {
  margin-top: 20px;
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
}

th, td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: center;
}

.error-message {
  color: red;
  font-size: 14px;
  margin-top: 10px;
}
</style>
