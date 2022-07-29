<template>
  <div>
    <p>state:</p>
    <pre>
      {{ state }}
    </pre>
    <a href="#" @click.prevent="logIn">Log In</a>
    <a href="#" @click.prevent="logOut">Log Out</a>
  </div>
</template>

<script lang="ts" setup>
type State = {
  greeting: string;
  counter: number;
  user: User;
};
interface User {
  id: number;
  first_name: string;
  last_name: string;
  email: string;
  phone: string;
  email_verified_at: string;
  passsword: string;
  is_admin: string;
  remember_token: string;
  organization_id: number;
  referral_url: string;
  utm_source: string;
  utm_medium: string;
  utm_campaign: string;
  utm_term: string;
  utm_content: string;
  created_at: string;
  updated_at: string;
  deleted_at: string;
}
const state = reactive<State>({
  greeting: 'test',
  counter: 0,
  user: {} as User,
});
// store a reactive reference to the value of the cookie
const token = useCookie('XSRF-TOKEN');
const logIn = async () => {
  // If no XSRF token cookie exists, Send the request to the API to get a new one.
  // useFetch will store the token value as a cookie (XSRF-TOKEN) in our browser via `credentials: true`
  // then, we will pass that value along as an `x-xsrf-token` request header
  if (!token.value) {
    await useFetch('http://localhost:8000/sanctum/csrf-cookie', { credentials: 'include' });
  }

  // send a request to the API to log in
  await useFetch('http://localhost:8000/login', {
    params: { email: 'jhutchcraft@lumenbuilder.com', password: '#Hutchy1134' },
    method: 'POST',
    credentials: 'include',
    headers: {
      accept: 'application/json',
      'x-requested-with': 'XMLHttpRequest',
      'x-xsrf-token': token.value,
    },
  });

  // Send a request to the API to get the user's data
  const {
    data,
    pending,
    error,
    refresh: refreshToken,
  } = await useFetch('http://localhost:8000/api/user', {
    params: {},
    method: 'GET',
    credentials: 'include',
    headers: {
      accept: 'application/json',
      'x-requested-with': 'XMLHttpRequest',
      'x-xsrf-token': token.value,
    },
  });

  Object.assign(state.user, data.value);
};

const logOut = async () => {
  try {
    if (!token.value) throw 'invalid token';
    const {
      data,
      pending,
      error,
      refresh: refreshToken,
    } = await useFetch('http://localhost:8000/logout', {
      params: {},
      method: 'POST',
      credentials: 'include',
      headers: {
        accept: 'application/json',
        'x-requested-with': 'XMLHttpRequest',
        'x-xsrf-token': token.value,
      },
    });
    state.user = {} as User;
    token.value = null;
    console.log(data);
  } catch (error) {
    console.error(error);
  }
};
</script>
