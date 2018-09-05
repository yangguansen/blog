---
title: 使用fetch代替ajax
date: 2018-09-04 09:51:09
tags:
	- JavaScript
---
基于fetch API，前后端交互不再依赖第三方包。
<!--more-->

> 引言

基于fetch API，前后端交互不再依赖第三方包。
以下是自己项目中使用fetch方法进行交互。

- get
```
const fetchGET = ( url, params = {} ) => {

  let headerObj = {
    credentials: 'include', //  设置跨域时，也可以发送cookie
    method: 'GET',
  };

  let urlObj = url;
  
  if ( JSON.stringify( params ) !== '{}' ) {
    let temp = [];
    const len = Object.keys( params ).length;
    Object.keys( params ).map( ( v, i ) => {
      temp.push( `${v}=${params[ v ]}` );
      if ( i + 1 < len ) {
        temp.push( '&' );
      }
    } );

    urlObj = `${url}?${temp.join( '' )}`;
  }

  return fetch( `${urlObj}`, headerObj )
    .then( ( response ) => {
      return response.json();
    } )
    .then( ( data ) => {
      if ( data.status < 0 ) {
        Message.error( data.msg || '请求失败' );
      } else if (data.status === 1){
        data.msg && Message.success( data.msg );
      }
      return data;
    } )
    .catch( ( e ) => {
      return {
        msg: '请求失败'
      }
    } )
}
```

- post file
使用formData进行文件提交。
```
const fetchFILE = ( url, params = {} ) => {
 
  let headerObj = {
    credentials: 'include',
    method: 'POST'
  };

  return fetch( url, {
    ...headerObj,
    body: params
  } ).then( function ( response ) {
    return response.json();
  } ).then(data => {
    if ( data.status < 0 ) {
      Message.error( data.msg || '请求失败' );
    } else if (data.status === 1){
      data.msg && Message.success( data.msg );
    }
    return data;
  }).catch( function ( e ) {
    return {
      msg: '请求失败'
    }
  } )
}
```

- post

```
const fetchPOST = ( url, params = {}, header = {} ) => {
 
  let body = [];

  if ( JSON.stringify( params ) !== '{}' ) {
    const len = Object.keys( params ).length;
    Object.keys( params ).map( ( v, i ) => {
      body.push( `${v}=${params[ v ]}` );
      if ( i + 1 < len ) {
        body.push( '&' )
      }
    } )
  }

  return fetch( url,
    {
      method: 'POST',
      credentials: 'include',
      headers: {
        'Content-Type': 'application/x-www-form-urlencoded'
      },
      body: body.join( '' )
    } )
    .then( ( response ) => {
      return response.json();
    } )
    .then( ( data ) => {
      if ( data.status < 0 ) {
        Message.error( data.msg || '请求失败' );
      } else if (data.status === 1){
        data.msg && Message.success( data.msg );
      }
      return data;
    } )
    .catch( ( e ) => {
      return {
        msg: '请求失败'
      }
    } )
}
```
