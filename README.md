# GraphQL WX-Cloud-Server Middleware（小程序云开发的graphql中间层）

## Installation

```sh
npm install --save wx-server-graphql
```

## Server Simple Example

Just mount `wx-server-graphql` as the `/graphql` handler:

graphql/index.js

```js
// 云函数入口文件
const cloud = require('wx-server-sdk')
const { graphqlWXServer } = require('wx-server-graphql')
var { buildSchema } = require('graphql')

// 使用 GraphQL Schema Language 创建一个 schema
var schema = buildSchema(`
  type Query {
    hello: String
  }
`)

// root 提供所有 API 入口端点相应的解析器函数
var root = {
	hello: () => {
		return 'Hello world!'
	},
}

cloud.init()

// 云函数入口函数
exports.main = async (event, context) =>
	await graphqlWXServer({
		wxParams: event,
		context,
		schema: schema,
		rootValue: root,
	})
```

## Client Simple Example
apolloProvider.js
```js
import React from 'react'
import { ApolloClient } from 'apollo-client'
import { ApolloProvider } from '@apollo/react-hooks'
import { InMemoryCache } from 'apollo-cache-inmemory'
import { ApolloLink, Observable } from 'apollo-link'
import { print } from 'graphql/language/printer'

// 利用link重置apolloClient请求grapql的方式为wx.cloud.callFunction
// 参考文档 https://www.apollographql.com/blog/apollo-link-creating-your-custom-graphql-client-c865be0ce059/
class WXLink extends ApolloLink {
  constructor(options = {}) {
    super()
    this.options = options
  }
  request(operation) {
    return new Observable(observer => {
      wx.cloud.callFunction({
        name: this.options.name || 'graphql',
        data: {
          ...operation,
          query: print(operation.query)
        },
        success: function(res) {
          observer.next(res.result)
          observer.complete()
        },
        fail: observer.error
      })
    })
  }
}

wx.cloud.init({
	env: 'env-id',
})

const client = new ApolloClient({
	link: new WXLink({
		name: 'graphql',
	}),
	cache: new InMemoryCache(),
})

export const Provider = ({ children }) => (
	<ApolloProvider client={client}>{children}</ApolloProvider>
)
```

## 具体使用细节可以参考文章
[《小程序云开发支持graphql》](https://developers.weixin.qq.com/community/develop/article/doc/000e2e48dbcdc05166aa1a97050813)
