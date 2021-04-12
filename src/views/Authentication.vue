<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title class="ion-text-center">{{
          mode === AuthMode.SignIn ? "Sign In" : "Sign Up"
        }}</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content>
      <ion-grid>
        <ion-row>
          <ion-col
            size-sm="12"
            size-md="8"
            offset-md="2"
            size-lg="6"
            offset-lg="3"
            size-xl="4"
            offset-xl="4"
          >
            <ion-card>
              <ion-card-header class="ion-text-center">
                <ion-card-title> Welcome to Photoshare! </ion-card-title>
                <ion-card-subtitle class="ion-margin-top">{{
                  mode === AuthMode.SignIn ? "Sign In" : "Sign Up"
                }}</ion-card-subtitle>
              </ion-card-header>
              <ion-card-content
                v-if="errorMessage"
                class="ion-text-center underlined"
              >
                {{ errorMessage }}
              </ion-card-content>
              <ion-card-content>
                <form
                  @submit.prevent="
                    mode === AuthMode.SignIn
                      ? signInWithEmailAndPassword(email, password)
                      : signUpWithEmailAndPassword(name, email, password)
                  "
                >
                  <ion-item v-if="mode === AuthMode.SignUp">
                    <ion-label position="floating">Name</ion-label>
                    <ion-input type="text" v-model="name"></ion-input>
                  </ion-item>
                  <ion-item>
                    <ion-label position="floating">Email</ion-label>
                    <ion-input type="text" v-model="email"></ion-input>
                  </ion-item>
                  <ion-item>
                    <ion-label position="floating">Password</ion-label>
                    <ion-input type="password" v-model="password"></ion-input>
                  </ion-item>
                  <ion-button
                    type="submit"
                    expand="block"
                    color="primary"
                    class="ion-margin-top"
                  >
                    {{ mode === AuthMode.SignIn ? "Sign In" : "Sign Up" }}
                  </ion-button>
                  <ion-button
                    expand="block"
                    color="secondary"
                    class="ion-margin-top"
                    @click="toggleAuthMode"
                  >
                    {{
                      mode === AuthMode.SignIn
                        ? "I Don't Have An Account"
                        : "Cancel"
                    }}
                  </ion-button>
                </form>
              </ion-card-content>
            </ion-card>
          </ion-col>
        </ion-row>
      </ion-grid>
    </ion-content>
  </ion-page>
</template>

<script lang='ts'>
import {
  IonPage,
  IonHeader,
  IonToolbar,
  IonTitle,
  IonContent,
  IonGrid,
  IonRow,
  IonCol,
  IonCard,
  IonCardHeader,
  IonCardSubtitle,
  IonCardTitle,
  IonCardContent,
  IonInput,
  IonButton,
  IonLabel,
  IonItem,
} from "@ionic/vue";

import { auth, firestore } from "../main";
import { reactive, toRefs } from "vue";
import { useRouter } from "vue-router";

enum AuthMode {
  SignIn,
  SignUp,
}

export default {
  name: "Authentication",
  components: {
    IonPage,
    IonHeader,
    IonToolbar,
    IonTitle,
    IonContent,
    IonGrid,
    IonRow,
    IonCol,
    IonCard,
    IonCardHeader,
    IonCardSubtitle,
    IonCardTitle,
    IonCardContent,
    IonInput,
    IonButton,
    IonLabel,
    IonItem,
  },
  setup() {
    const router = useRouter();
    const state = reactive({
      name: "",
      email: "",
      password: "",
      mode: AuthMode.SignIn,
      errorMessage: "",
    });

    const signInWithEmailAndPassword = async (
      email: string,
      password: string
    ) => {
      try {
        if (!email || !password) {
          state.errorMessage = "Please enter an email and a password.";
          return;
        }

        await auth.signInWithEmailAndPassword(email, password);
        router.push("/tabs/tab1");
      } catch (err) {
        state.errorMessage = err.message;
      }
    };

    const signUpWithEmailAndPassword = async (
      name: string,
      email: string,
      password: string
    ) => {
      try {
        if (!name || !email || !password) {
          state.errorMessage = "Please enter a name, email, and password.";
        }

        const authResponse = await auth.createUserWithEmailAndPassword(
          email,
          password
        );

        const uid = authResponse.user?.uid;
        await firestore.collection("users").doc(uid).set({
          name,
          email,
        });

        router.push("/tabs/tab1");
      } catch (err) {
        state.errorMessage = err.message;
      }
    };

    const toggleAuthMode = () => {
      state.mode =
        state.mode === AuthMode.SignIn ? AuthMode.SignUp : AuthMode.SignIn;
    };

    return {
      ...toRefs(state),
      signInWithEmailAndPassword,
      signUpWithEmailAndPassword,
      AuthMode,
      toggleAuthMode,
    };
  },
};
</script>

<style scoped>
.center {
  display: flex;
  height: 90vh;
  width: 100%;
  align-items: center;
  justify-content: center;
}
</style>
