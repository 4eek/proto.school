<template>
  <div class="lesson-blog-03">
    <Lesson v-bind:text="text" v-bind:code="code"
            :validate="validate"
            :exercise="exercise"
            lessonTitle="Multiple links">
    </Lesson>
  </div>
</template>

<script>
import Lesson from '../../components/Lesson'
import text from './03.md'
import exercise from './03-exercise.md'
import utils from './utils.js'
const shallowEqualArrays = require('shallow-equal/arrays')
const CID = require('cids')

const code = `/* globals ipfs */

const run = async () => {
  const natCid = await ipfs.dag.put({author: "Nat"})
  const samCid = await ipfs.dag.put({author: "Sam"})
  const treePostCid = await ipfs.dag.put({
    content: "trees",
    author: {"/": samCid.toBaseEncodedString()},
    tags: ["outdoor", "hobby"]
  })
  const computerPostCid = await ipfs.dag.put({
    content: "computers",
    author: {"/": natCid.toBaseEncodedString()},
    tags: ["hobby"]
  })

  // Add your code here
}

return run`

// eslint-disable-next-line no-unused-vars
const _solution = `
/* globals ipfs */

const run = async () => {
  const natCid = await ipfs.dag.put({author: "Nat"})
  const samCid = await ipfs.dag.put({author: "Sam"})
  const treePostCid = await ipfs.dag.put({
    content: "trees",
    author: {"/": samCid.toBaseEncodedString()},
    tags: ["outdoor", "hobby"]
  })
  const computerPostCid = await ipfs.dag.put({
    content: "computers",
    author: {"/": natCid.toBaseEncodedString()},
    tags: ["hobby"]
  })

  const outdoorTagCid = await ipfs.dag.put({
    tag: "outdoor",
    posts: [
      {"/": treePostCid.toBaseEncodedString()}
    ]
  })
  const hobbyTagCid = await ipfs.dag.put({
    tag: "hobby",
    posts: [
      {"/": treePostCid.toBaseEncodedString()},
      {"/": computerPostCid.toBaseEncodedString()}
    ]
  })

  return [outdoorTagCid, hobbyTagCid]
}

return run
`

const validate = async (result, ipfs) => {
  if (!result) {
    return {fail: 'You forgot to return a result :)'}
  }
  const validatedArray = utils.validateArrayOfCids(result, 2)
  if (validatedArray.fail) {
    return validatedArray
  }
  for (const cid of result) {
    const obj = await ipfs.dag.get(cid)
    const node = obj.value
    if (node.tag === undefined) {
      return {fail: 'Tag nodes need to have a `tag` field.'}
    }
    if (typeof node.tag !== 'string') {
      return {fail: '`tag` field needs to be a string.'}
    }
    if (node.posts === undefined) {
      return {fail: 'Tag nodes need to have a `posts` field.'}
    }
    if (!Array.isArray(node.posts)) {
      return {fail: 'The value of the `posts` field must be an array of links.'}
    }
    const isLinks = node.posts.every((post) => '/' in post)
    if (!isLinks) {
      return {fail: 'The values of the `posts` array must be links.'}
    }

    const treePostCid = 'zdpuAri55PR9iW239ahcbnfkFU2TVyD5iLmqEFmwY634KZAJV'
    const computerPostCid = 'zdpuAqaHPSosSZFRPe7u5q3yNqgg4JuvrLaUJxGamNPLhWivX'
    let expectedPosts
    switch (node.tag) {
      case 'hobby':
        expectedPosts = [treePostCid, computerPostCid]
        break
      case 'outdoor':
        expectedPosts = [treePostCid]
        break
      default:
        return {fail: `Wrong tag (${node.tag}). Did you mean "hobby" or "outdoor"?`}
    }
    const nodePosts = node.posts.map((post) => new CID(post['/']).toBaseEncodedString())
    if (!shallowEqualArrays(nodePosts.sort(), expectedPosts.sort())) {
      return {fail: `The posts of the tag "${node.tag}" ${utils.stringify(nodePosts)} did not match the the expected posts ${utils.stringify(expectedPosts)}.`}
    }
  }
  // Don't check the CIDs as then the order of the links within the tags would
  // matter. But that order really doesn't matter.
  return {success: 'All works!'}
}

export default {
  components: {
    Lesson
  },
  data: () => {
    return {
      code,
      text,
      validate,
      exercise
    }
  }
}
</script>
