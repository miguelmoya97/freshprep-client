<script setup>
import { ref, onMounted } from 'vue';
import { faker } from '@faker-js/faker';

const id = ref('UUID')
const users = ref([])
const RETRY_LIMIT = 50

const createUser = async () => {
  const { person, internet } = faker
  const firstName = person.firstName()
  const lastName = person.lastName()
  const email = internet.email({firstName, lastName})

  let retry = 0
  while(retry < RETRY_LIMIT) {
    try {
      const postResp = await fetch('http://localhost:3000/users', {
        method: 'POST',
        headers: {
          "Content-Type": "application/json"
        },
        body: JSON.stringify( { name: `${firstName} ${lastName}`, email } )
      })
      if (postResp.ok) {
        return;
      } else {
        throw new Error(postResp.statusText)
      }
    } catch (err) {
      console.error("Error: ", err)
      retry++
      setTimeout(()=>{},100)
    }
  }
}

const getUsers = async () => {
  let retry = 0
    while(retry < RETRY_LIMIT) {
      try {
        const resp = await fetch('http://localhost:3000/users?id=*');
        if (!resp.ok) {
          throw new Error(postResp.statusText)
        }
        const data = await resp.json()
        users.value = data
        return;
      } catch (err) {
        console.error("Error: ", err)
        setTimeout(()=>{},100)
        retry++
      }
  }
}

onMounted(async () => {
  await createUser()
  await getUsers()
})

const fetchUniqueId = async () => {
  const resp = await fetch('http://localhost:3000/ids');
  const data = await resp.json()
  id.value = data.id
}
</script>

<template>
  <main>
    <button v-on:click="fetchUniqueId">Generate Unique ID</button>
    <h3>{{ id }}</h3>
    <h1>Users</h1>
    <ul>
      <li v-for="user in users" :key="user.id">
        {{ user.name}}, {{user.email}}
      </li>
    </ul>
  </main>
</template>

<style scoped>
header {
  line-height: 1.5;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}
</style>
