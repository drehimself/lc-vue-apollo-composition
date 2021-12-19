<template>
  <div class="home">
    <h2>Queries</h2>
    <div v-if="loading">Loading...</div>
    <div v-else-if="error">{{ error.message }}</div>
    <div v-else>
      <ul v-if="result">
        <li v-for="post in posts.data" :key="post.id">
          {{ post.title }}
        </li>
      </ul>
    </div>

    <h2>Query Variables</h2>
    <div v-if="postResult">{{ postResult.post.title }}</div>

    <h2>Mutations</h2>
    <div>{{ createPostError }}</div>
    <form action="#" @submit.prevent="createPostMethod">
      <div>
        <label for="title">Title</label>
        <input v-model="title" type="text" id="title" />
      </div>
      <div>
        <label for="body">Body</label>
        <textarea
          v-model="body"
          name="body"
          id="body"
          cols="30"
          rows="10"
        ></textarea>
      </div>
      <div>
        <button>Create Post</button>
        <div v-if="createPostLoading">Loading...</div>
      </div>
    </form>
  </div>
</template>

<script>
import { useMutation, useQuery, useResult } from '@vue/apollo-composable'
import gql from 'graphql-tag'
import { ref, watchEffect } from 'vue'

export default {
  name: 'Home',
  setup() {
    const title = ref('')
    const body = ref('')
    const createPostError = ref('')

    const { result, loading, error } = useQuery(gql`
      query getPosts {
        posts {
          data {
            id
            title
          }
        }
      }
    `)

    const posts = useResult(result, null, data => data.posts)

    const { result: postResult } = useQuery(
      gql`
        query getPost($id: ID!) {
          post(id: $id) {
            title
            body
          }
        }
      `,
      {
        id: 1,
      }
    )

    // watchEffect(() => console.log(result.value))

    const {
      mutate: createPost,
      loading: createPostLoading,
      onError,
      onDone,
    } = useMutation(gql`
      mutation createPost($user_id: ID!, $title: String!, $body: String!) {
        createPost(user_id: $user_id, title: $title, body: $body) {
          id
          title
        }
      }
    `)

    onError(error => {
      const key = Object.keys(error.graphQLErrors[0].extensions.validation)[0]
      createPostError.value =
        error.graphQLErrors[0].extensions.validation[key][0]
    })

    onDone(result => {
      title.value = ''
      body.value = ''
      createPostError.value = ''
    })

    function createPostMethod() {
      createPost({
        user_id: 1,
        title: title.value,
        body: body.value,
      })
    }

    return {
      title,
      body,
      createPostError,
      result,
      posts,
      loading,
      error,
      postResult,
      createPostMethod,
      createPostLoading,
    }
  },
}
</script>
