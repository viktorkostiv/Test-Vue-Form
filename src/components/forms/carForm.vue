<template>
  <div class="app-wrapper">
    <simple-card class="vd-form">
      <template #title>Autoverzekering vergelijken</template>

      <template #content>
        <Form @submit="onSubmit">
          <!-- LicensePlate -->
          <p class="label">License Plate</p>
          <Field
            class="baseInput uppercase"
            name="licensePlate"
            type="licensePlate"
            :rules="validateLicensePlate"
            placeholder="XXXXXX"
          />
          <ErrorMessage name="licensePlate" class="error-message" />
          <!-- Zipcode -->
          <p class="label">Zip Code</p>
          <Field
            class="baseInput uppercase"
            name="zipCode"
            type="zipCode"
            placeholder="XXXX99"
            :rules="validateZipCode"
          />
          <ErrorMessage name="zipCode" class="error-message" />
          <!-- Housenumber -->
          <p class="label">House Number</p>
          <Field
            class="baseInput"
            name="houseNumber"
            type="houseNumber"
            :rules="validateHouseNumber"
          />
          <ErrorMessage name="houseNumber" class="error-message" />
          <!-- Housenumber addition -->
          <p class="label">House Number Addition <span>Optional</span></p>
          <Field
            class="baseInput"
            name="houseNumberAddition"
            type="houseNumberAddition"
            :rules="validateHouseNumberAdd"
          />
          <ErrorMessage name="houseNumberAddition" class="error-message" />
          <!-- birthdate -->
          <p class="label">Birth Date</p>
          <Field
            class="baseInput"
            name="birthDate"
            type="birthDate"
            v-slot="{ field }"
            :rules="validateBirthDate"
          >
            <Datepicker v-bind="field" class="baseInput" typeable placeholder="DD-MM-YYYY" />
          </Field>
          <ErrorMessage name="birthDate" class="error-message" />
          <!-- ClaimFree years -->
          <p class="label">ClaimFree Years</p>
          <Field
            class="baseInput"
            name="claimFreeYears"
            type="claimFreeYears"
            as="select"
            v-slot="{ value }"
            v-model="claimFreeYear"
          >
            <option v-for="year in claimFreeYears" :key="year" :value="year" :selected="value">
              {{ year }}
            </option>
          </Field>
          <!-- Kilometrage -->
          <p class="label">Kilometrage</p>
          <Field
            class="baseInput"
            name="kilometrage"
            type="kilometrage"
            as="select"
            v-slot="{ value }"
            v-model="currentKilometrage"
          >
            <option
              v-for="kilometrage in kilometrageList"
              :key="kilometrage"
              :value="kilometrage"
              :selected="value"
            >
              {{ kilometrage }}
            </option>
          </Field>
          <base-button>Vergelijken</base-button>
        </Form>
      </template>
    </simple-card>
    <simple-card class="vd-form" v-if="(carBrand != null && manufacturingYear != null)">
      <template #title>Car Information</template>
      <template #content>
        <p>Car Brand: <strong>{{ carBrand }}</strong></p>
        <p>Manufacturing Year: <strong>{{ manufacturingYear.substr(0, 4) }}</strong></p>
      </template>
    </simple-card>
  </div>
</template>

<script>
import { Options, Vue } from "vue-class-component";
import { Form, Field, ErrorMessage } from "vee-validate";
import Datepicker from "vue3-datepicker";
import SimpleCard from "../ui/simpleCard.vue";
import BaseButton from "../ui/baseButton.vue";
@Options({
  components: {
    SimpleCard,
    BaseButton,
    Form,
    Field,
    ErrorMessage,
    Datepicker,
  },
})
export default class CarForm extends Vue {
  data() {
    return {
      userAge: 0,
      claimFreeYear: 0,
      currentKilometrage: "0 t/m 7500 KM",
      carBrand: null,
      manufacturingYear: null,
      kilometrageList: [
        "0 t/m 7500 KM",
        "7501 t/m 10000 KM",
        "10001 t/m 12000 KM",
        "12001 t/m 15000 KM",
        "15000 t/m 20000 KM",
        "20001 t/m 25000 KM",
        "25001 t/m 30000 KM",
        "30001 t/m 90000 KM",
      ],
      currentBranch: "dev",
      comments: null,
      fileUrl: "https://opendata.rdw.nl/resource/m9d7-ebf2.json?",
    };
  }

  // Computed Properties

  get claimFreeYears() {
    return [...Array(this.userAge + 1).keys()];
  }

  // Button Actions

  async onSubmit(values) {
    const response = await fetch(
      `https://opendata.rdw.nl/resource/m9d7-ebf2.json?kenteken=${values.licensePlate.toUpperCase()}`
    );

    const responseData = await response.json();

    if (!response.ok) {
      const error = new Error(responseData.message || "Failed to fetch!");
      throw error;
    }

    this.carBrand = responseData[0].merk;
    this.manufacturingYear = responseData[0].datum_eerste_toelating;

    const newLink = new URLSearchParams({
      licensePlate: values.licensePlate,
      zipCode: values.zipCode,
      houseNumber: values.houseNumber,
      houseNumberAddition: values.houseNumberAddition,
      birthDate: values.birthDate,
      claimFreeYears: values.claimFreeYears,
      kilometrage: values.kilometrage,
    }).toString();

    console.log(newLink);
  }

  // Validations

  validateLicensePlate(value) {
    if (!value || !value.length) {
      return "This field is required";
    }
    const regex = /^[a-zA-Z0-9]{6}$/;
    if (!regex.test(value)) {
      return "This field must be a valid License Plate";
    }
    return true;
  }

  validateZipCode(value) {
    if (!value || !value.length) {
      return "This field is required";
    }
    const regex = /^[a-zA-Z0-9]{6}$/;
    if (!regex.test(value) || value.substr(-2) != "99") {
      return "This field must be a valid Zip Code";
    }
    return true;
  }

  validateHouseNumber(value) {
    if (!value || !value.length) {
      return "This field is required";
    }
    const regex = /^[0-9]*$/;
    if (!regex.test(value)) {
      return "This field must be a valid House Number";
    }
    return true;
  }

  validateHouseNumberAdd(value) {
    const regex = /^[0-9]*$/;
    if (regex.test(value) || !value || !value.length) {
      return true;
    }
    return "This field must be a valid House Number";
  }

  validateBirthDate(value) {
    const date = new Date();
    let age = (date - value) / 31557600000;
    if (age > 100 || age < 18 || !age) {
      return "This field must be a valid Birth Date";
    }

    this.userAge = Math.floor(age) - 18;
    return true;
  }
}
</script>

<style>
.vd-form {
  width: 330px;
  height: fit-content;
}
.app-wrapper {
  margin: 50px auto;
  justify-content: space-between;
  width: 700px;
  display: flex;
}
.label {
  margin-bottom: 0px;
  width: 100%;
  display: flex;
}
.label span {
  color: #a3a3a3;
  font-size: 14px;
  font-style: italic;
  margin-left: auto;
}
input.baseInput.uppercase {
  text-transform: uppercase;
}
select,
input.baseInput {
  margin: 5px 0 10px;
  width: -webkit-fill-available;
  padding: 5px 14px;
  box-shadow: 0 5px 40px rgb(112 144 176 / 20%);
  border-radius: 5px;
  border: 1px solid #c4c4c4;
  outline: none;
  transition: all 0.3s ease;
}
input.baseInput:focus-visible {
  border-color: #0cbe3b;
}
.error-message {
  margin-top: 5px;
  font-size: 14px;
  font-weight: 600;
  color: #c51e1e;
}
@media only screen and (max-width: 768px) {
  .vd-form {
    width: 100%;
  }
}
</style>
