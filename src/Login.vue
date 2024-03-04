<template>
  <div v-if="!accessToken" class="auth-container">
    <div class="auth-box">
      <img class="phe-login-img" width="250px"
        src="https://storage.googleapis.com/prod-phoneemail-prof-images/phem-widgets/phe-signin-box.svg"
        alt="phone email login demo" />
      <h1 style="margin: 10px;">Sign In</h1>
      <p style="color: #a6a6a6;">Welcome to Sign In with Phone</p>
      <button @click="openAuthWindow" class="auth-button">
        <img src="https://storage.googleapis.com/prod-phoneemail-prof-images/phem-widgets/phem-phone.svg"
          alt="phone email" style="margin-right: 10px;" />
        Sign In with Phone
      </button>
    </div>
  </div>

  <div v-else>
    <div class="auth-container">
      <div class="auth-box">
        <img class="phe-login-img" width="250px"
          src="https://storage.googleapis.com/prod-phoneemail-prof-images/phem-widgets/phe-signin-success.svg"
          alt="phone email login demo" />
        <div class="phem-card-body">
          <h1>Welcome!</h1>
          <h4 style="line-height: 36px;">You are logged in successfully with <br /> {{ userDetails.countryCode }} {{
            userDetails.phoneNo }}</h4>
        </div>
        <button @click="handleBack" class="auth-button">Back</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      accessToken: null,
      CLIENT_ID: "**************************",
      userDetails: {
        countryCode: "",
        phoneNo: "",
      },
    };
  },
  mounted() {
    const searchParams = new URLSearchParams(window.location.search);
    this.accessToken = searchParams.get('access_token');

    if (this.accessToken) {
      this.httpRequest();
    }
  },
  methods: {
    openAuthWindow() {
      const REDIRECT_URL = window.location.href;
      const AUTH_URL = `https://auth.phone.email/log-in?client_id=${this.CLIENT_ID}&redirect_url=${REDIRECT_URL}`;
      window.open(AUTH_URL, 'peLoginWindow', 'toolbar=0,scrollbars=0,location=0,statusbar=0,menubar=0,resizable=0,width=500,height=560,top=' + (window.screen.height - 600) / 2 + ',left=' + (window.screen.width - 500) / 2);
    },
    async httpRequest() {
      try {
        const url = "https://eapi.phone.email/getuser";
        const data = new FormData();

        data.append("access_token", this.accessToken);
        data.append("client_id", this.CLIENT_ID);

        const response = await fetch(url, { method: "POST", body: data });

        if (!response.ok) {
          throw new Error("Network response was not ok");
        }

        const responseData = await response.json();

        if (responseData.status !== 200) {
          throw new Error("Something went wrong");
        }

        this.userDetails = {
          countryCode: responseData.country_code,
          phoneNo: responseData.phone_no,
        };

        // Set cookie with 1-day expiration
        const cookieExpire = new Date(Date.now() + 1 * 24 * 60 * 60 * 1000).toUTCString();
        document.cookie = `ph_email_jwt=${responseData.ph_email_jwt}; expires=${cookieExpire}; path=/`;
      } catch (error) {
        console.error("Fetch error:", error);
      }
    },
    handleBack() { window.location.href = "/" },
  },
};
</script>

<style scoped>
body {
  background-color: #f1f5f9;
}

.auth-container {
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 50px 30px;
}

.auth-box {
  color: #024430 !important;
  text-align: center;
  background-color: #fff;
  padding: 30px;
  border-radius: 0.5rem;
  box-shadow: 0 1px 3px rgba(17, 24, 39, .09);
  width: 100%;
  max-width: 420px;
  margin: 0 auto;
  font-family: sans-serif, serif, system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
  line-height: 28px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.auth-button {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 14px 20px;
  background-color: #02BD7E;
  font-weight: bold;
  color: #ffffff;
  border: none;
  border-radius: 3px;
  font-size: inherit;
  cursor: pointer;
  max-width: 320px;
  width: 100%;
}</style>
