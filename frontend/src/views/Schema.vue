<!--
 Copyright (c) 2020 - for information on the respective copyright owner
 see the NOTICE file and/or the repository at
 https://github.com/hyperledger-labs/organizational-agent

 SPDX-License-Identifier: Apache-2.0
-->
<template>
  <v-container justify-center>
    <v-card v-if="!isLoading" class="mx-auto">
      <v-card-title class="bg-light">
        <v-btn depressed color="secondary" icon @click="$router.go(-1)">
          <v-icon dark>$vuetify.icons.prev</v-icon>
        </v-btn>
        <span>{{ data.label }}</span>
        <v-layout align-end justify-end>
          <!-- <v-btn depressed icon @click="isUpdatingName = !isUpdatingName">
                    <v-icon dark>$vuetify.icons.pencil</v-icon>
                </v-btn> -->
          <v-btn
            depressed
            color="red"
            icon
            :disabled="data.isReadOnly"
            @click="deleteSchema"
          >
            <v-icon dark>$vuetify.icons.delete</v-icon>
          </v-btn>
        </v-layout>
      </v-card-title>
      <v-container>
        <v-list-item class="mt-4">
          <v-list-item-title
            class="grey--text text--darken-2 font-weight-medium"
          >
            Schema ID:
          </v-list-item-title>
          <v-list-item-subtitle>
            {{ data.schemaId }}
          </v-list-item-subtitle>
        </v-list-item>

        <h4 class="my-4 grey--text text--darken-3">Schema Attributes</h4>

        <v-list-item
          v-for="attribute in data.schemaAttributeNames"
          :key="attribute.id"
        >
          <p class="grey--text text--darken-2 font-weight-medium">
            {{ attribute }}
          </p>
        </v-list-item>

        <trusted-issuers :id="id" :trustedIssuers="trustedIssuers" />
      </v-container>
      <v-card-actions>
        <v-layout align-end justify-end>
          <v-bpa-button color="primary" :to="{ name: 'SchemaSettings' }"
            >Close</v-bpa-button
          >
        </v-layout>
      </v-card-actions>
    </v-card>
  </v-container>
</template>

<script>
import { EventBus } from "../main";
import TrustedIssuers from "../components/TrustedIssuers";
import VBpaButton from "@/components/BpaButton";
export default {
  name: "Schema",
  props: {
    id: String, //schema ID
    schema: Object,
  },
  components: {
    VBpaButton,
    TrustedIssuers,
  },
  mounted() {
    EventBus.$emit("title", "Schema");
    console.log("SCHEMA", this.schema);
    this.fetch();
  },
  data: () => {
    return {
      data: [],
      isLoading: true,
      trustedIssuers: [],
    };
  },
  computed: {},
  methods: {
    fetch() {
      this.isLoading = true;
      this.$axios
        .get(`${this.$apiBaseUrl}/admin/schema/${this.id}`)
        .then((result) => {
          console.log(result);
          if ({}.hasOwnProperty.call(result, "data")) {
            this.data = result.data;
            // Init trusted issuers
            if ({}.hasOwnProperty.call(this.data, "trustedIssuer")) {
              this.trustedIssuers = this.data.trustedIssuer;
            }
            this.isLoading = false;
          }
        })
        .catch((e) => {
          this.isLoading = false;
          if (e.response.status === 404) {
            this.data = [];
          } else {
            console.error(e);
            EventBus.$emit("error", e);
          }
        });
    },
    deleteSchema() {
      this.$axios
        .delete(`${this.$apiBaseUrl}/admin/schema/${this.id}`)
        .then((result) => {
          console.log(result);
          if (result.status === 200) {
            EventBus.$emit("success", "Schema deleted");
            this.$router.push({
              name: "SchemaSettings",
            });
          }
        })
        .catch((e) => {
          console.error(e);
          EventBus.$emit("error", e);
        });
    },
  },
};
</script>

<style scoped>
.bg-light {
  background-color: #fafafa;
}

.bg-light-2 {
  background-color: #ececec;
}
</style>
