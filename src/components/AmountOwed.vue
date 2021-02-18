<template>
    <div>
      <h3>AMOUNT OWED {{ this.amountOwed }}  &nbsp;
        <b-button style="display: inline" variant="primary" size="lg" @click="$emit('complete-checkout')">
        <!--complete-checkout is in ValetView component-->
            PAID
        </b-button>
      </h3>
          <!--
          **The cancel button requires more revision of the
          website flow:
          *car would need to be checked back in; better to change car status and change status back at cancellation then fully check car out on paid button click
          <b-col sm="3">
            <b-button @click="onCancel, $emit('click-cancel')" variant="danger" class="h-75">Cancel</b-button>
        </b-col>
        -->
    
     </div>
</template>

<script>
export default {
    name: "amount-owed",
    props: ["amountOwed"]
}
</script>