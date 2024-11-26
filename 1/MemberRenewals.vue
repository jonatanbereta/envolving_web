<script setup lang="ts">
import { defineProps, ref } from "vue";
import { MemberData } from "@/models/IMemberData";
import Lucide from "@/base-components/Lucide";
import { dateTodayYMD, oneYearAfterTodayYMD } from "@/helpers/handlers";
import Table from "@/base-components/Table";
import { convertDate } from "@/helpers/masks";
import { isDatePast } from "@/helpers/validations";
import Button from "@/base-components/Button";
import PaymentRegister from "@/components/MemberPayments/PaymentRegister.vue";
import ServiceConfigs from "@/services/ServiceConfigs";
import { ProgramsEnum } from "@/enums/programs.enum";
import { Program } from "@/models/IProgram";
import MemberRenewalsService from "@/services/MemberRenewalsService";
import { useDashboardStore } from "@/stores/dashboard-store";

const props = defineProps<{
  member: MemberData;
}>();

const emits = defineEmits(["close"]);

const configService = new ServiceConfigs();
const memberRenewalService=new MemberRenewalsService();

const dashboardStore=useDashboardStore();

const message = ref({
  message: "",
  type: "",
});

const showPaymentRegister = ref(false);
const price = ref(0);

async function inscriptionVerify() {
  await getPrice();

  if (isDatePast(props.member.next_renewal)) {
    message.value.message =
      "Le membre est encore actif. Aucune action nécessaire.";
    message.value.type = "warning";
    setTimeout(() => {
      message.value.message = "";
      message.value.type = "";
    }, 5000);
    emits("close");
  } else {
    showPaymentRegister.value = true;
  }
}

const getPrice = async () => {
  await configService
    .getPrograms()
    .then((response) => {
      price.value = response.find(
        (program: Program) => program.name === ProgramsEnum.BA
      ).year_cost;
    })
    .catch((error) => {
      console.error("Failed to fetch price:", error);
    });
};

async function register() {

  const renewalData={
    member_id:props.member.id,
    amount_paid:price.value,
    inscription_date:dateTodayYMD(),
    expiration:oneYearAfterTodayYMD(),
  }

await memberRenewalService.postRenewal(renewalData).then((response) => {
    showPaymentRegister.value = false;
    message.value.message = "Renouvellement effectué avec succès.";
    message.value.type = "success";
    dashboardStore.renewals++;
    setTimeout(() => {
      message.value.message = "";
      message.value.type = "";
    }, 5000);
    emits("close",props.member.id);
  })
  .catch((error) => {
    console.error("Failed to create renewal:", error);
  });
}
</script>
<template>
  <div class="grid grid-cols-12 gap-2 box px-3 py-5">
    <div class="col-span-12">
      <h2 class="text-primary text-center uppercase font-bold mb-4">
        Historique de Renouvellements
      </h2>
    </div>
    <div class="col-span-12">
      <Table class="">
        <Table.Thead>
          <Table.Tr>
            <Table.Th class="text-xs border-b-0 whitespace-nowrap">
              INSCRIPTION
            </Table.Th>
            <Table.Th class="text-xs border-b-0 whitespace-nowrap">
              EXPIRATION
            </Table.Th>
            <Table.Th class="text-xs border-b-0 whitespace-nowrap">
              MONTANT PAYÉE
            </Table.Th>
            <Table.Th class="text-xs text-center border-b-0 whitespace-nowrap">
              SITUATION
            </Table.Th>
          </Table.Tr>
        </Table.Thead>
        <Table.Tbody>
          <Table.Tr
            v-for="renewal in member.renewals"
            :key="renewal.id"
            class="intro-x"
          >
            <Table.Td
              class="first:rounded-l-md last:rounded-r-md whitespace-nowrap bg-white border-b-0 dark:bg-darkmode-600"
            >
              {{ convertDate(renewal.inscription_date) }}
            </Table.Td>
            <Table.Td
              class="first:rounded-l-md last:rounded-r-md whitespace-nowrap bg-white border-b-0 dark:bg-darkmode-600"
            >
              {{ convertDate(renewal.expiration) }}
            </Table.Td>

            <Table.Td
              class="first:rounded-l-md last:rounded-r-md whitespace-nowrap bg-white border-b-0 dark:bg-darkmode-600"
              ><span>${{ renewal.amount_paid }}</span>
            </Table.Td>
            <Table.Td
              class="first:rounded-l-md last:rounded-r-md whitespace-nowrap bg-white border-b-0 dark:bg-darkmode-600"
            >
              <p
                v-if="isDatePast(renewal.expiration)"
                class="bg-success font-bold text-white px-2 py-1 text-xs rounded-xl text-center"
              >
                Actif
              </p>

              <p
                v-else
                class="bg-danger font-bold text-white px-2 py-1 text-xs rounded-xl text-center"
              >
                Expiré
              </p>
            </Table.Td>
          </Table.Tr>
        </Table.Tbody>
      </Table>
    </div>
    <div class="col-span-12 mt-4 text-center">
      <div
        v-if="message.message"
        class="mb-4 p-5 border font-bold rounded-lg"
        :class="[
          message.type == 'success' && 'text-success border-success',
          message.type == 'warning' && 'text-pending border-pending',
          message.type == 'danger' && 'text-danger border-danger',
        ]"
      >
        {{ message.message }}
      </div>
      <Button v-if="!showPaymentRegister" @click="inscriptionVerify" variant="primary">
        Ajouter un renouvellement
      </Button>
    </div>

    <div v-if="showPaymentRegister" class="col-span-12 mt-4 px-4 pb-4 box">
      <PaymentRegister
        :price="price"
        :member-id="member.id"
        :member-program="member.program_name"
        :label="'Le paiement a-t-il été effectué?'"
        @paid="register"
        @cancel="showPaymentRegister = false"
      />
    </div>
  </div>
</template>
