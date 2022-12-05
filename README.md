# node-sample

Single-Route, Multi-Origin Proxy (1)
There may be some circumstances where we need to send data from another service via our service. In these cases, we could actually use an HTTP request library like got as explored in the prior chapter. However, using a proxying library is a viable alternative that provides a more configuration-based approach vs the procedural approach of using a request library.

Let's start by defining a route that will take a querystring parameter called url and then respond from whatever URL is specified in that parameter.

Let's initialize a new Fastify project:

node -e "fs.mkdirSync('my-route-proxy')"
cd my-route-proxy
npm init fastify

Now let's install the fastify-reply-from:

npm install fastify-reply-from

Then create a plugins/reply-from.js file with the following contents:

'use strict'

const fp = require('fastify-plugin')

module.exports = fp(async function (fastify, opts) {
  fastify.register(require('fastify-reply-from'), {
    errorHandler: false
  })
})

'use strict'
const { Readable } = require('stream')
async function * upper (res) {
  for await (const chunk of res) {
    yield chunk.toString().toUpperCase()
  }
}
module.exports = async function (fastify, opts) {
  fastify.get('/', async function (request, reply) {
    const { url } = request.query
    try {
      new URL(url)
    } catch (err) {
      throw fastify.httpErrors.badRequest()
    }
    return reply.from(url, {
      onResponse (request, reply, res) {
        reply.send(Readable.from(upper(res)))
      }
    })
  })
}


'use strict'
const proxy = require('fastify-http-proxy')
module.exports = async function (fastify, opts) {
  fastify.register(proxy, {
    upstream: 'htt‌ps://news.ycombinator.com/'
  })
}

'use strict'

const proxy = require('fastify-http-proxy')
const sensible = require('fastify-sensible')
module.exports = async function (fastify, opts) {
  fastify.register(sensible)
  fastify.register(proxy, {
    upstream: 'https://news.ycombinator.com/',
    async preHandler(request, reply) {
      if (request.query.token !== 'abc') {
        throw fastify.httpErrors.unauthorized()
      }
    }
  })

}
