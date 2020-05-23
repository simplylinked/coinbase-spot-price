const url = "https://api.coinbase.com/v2/prices/{PAIR}/spot";
const type = 'application/json;charset=UTF-8'
const API_SECRET = "API_SECRET";
const API_SECRET_IN_HEADER = false;

async function handleRequest(request) {

  // response headers
  const responseInit = {
    headers: {
      'content-type': type
    }
  }

  // get JSON body
  const requestBody = await request.json()

  // get API_SECRET from header first
  var apiSecret = request.headers.get(API_SECRET);

  var responseJSON;

  // Check whether API_SECRET is configured
  if((apiSecret == null || apiSecret.length == 0) && API_SECRET_IN_HEADER) {
    responseJSON = {
        jobRunID: requestBody.id,
        status: "errored",
        error: 'missing API_SECRET',
    }

    return new Response(JSON.stringify(responseJSON), responseInit)
  }
  else {

    // get API_SECRET from body, which is populated by Chainlink node
    if(requestBody.data != null && requestBody.data.headers != null) {
      apiSecret = requestBody.data.headers.API_SECRET
    }
  
    // API_SECRET_IN_HEADER is false, if apiSecret not found in body, throw error
    if((apiSecret == null || apiSecret.length == 0) && !API_SECRET_IN_HEADER) {
      responseJSON = {
          jobRunID: requestBody.id,
          status: "errored",
          error: 'missing API_SECRET',
      }
      return new Response(JSON.stringify(responseJSON), responseInit)
    }  
    else {

      // request header for data provider
      const adapterRequestInit = {
          headers: {
            'content-type': type,
            'Authorization': 'Bearer ' + apiSecret
          }
        }

        // get crypto pair
        const pair = requestBody.data.pair

        // crypto pair must not be blank
        if(pair != null && pair.length > 0) {

          // the actual URL after crypto pair is filled  
          var tempUrl = url.replace("{PAIR}", pair)

          // May support multiple endpoints in the future
          const adapterResponses = await Promise.all([fetch(tempUrl, adapterRequestInit)])
          const adapterResults = await Promise.all([gatherResponse(adapterResponses[0])])  

          // Check whether the adapter return is valid
          if(adapterResults != null && adapterResults.length > 0 && adapterResults[0].hasOwnProperty("data")) {
            responseJSON = {
                jobRunID: requestBody.id,
                data: adapterResults[0].data,
            }
          }
          else {
            responseJSON = {
                jobRunID: requestBody.id,
                status: "errored",
                error: 'unsupported crypto pair',
            }
          }

        }
        else {
          responseJSON = {
                jobRunID: requestBody.id,
                status: "errored",
                error: 'missing crypto pair',
            }
        }

        return new Response(JSON.stringify(responseJSON), responseInit)
    }
  }
}

addEventListener('fetch', event => {
  return event.respondWith(handleRequest(event.request))
})

/**
 * gatherResponse awaits and returns a response body as a string.
 * Use await gatherResponse(..) in an async function to get the response body
 * @param {Response} response
 */
async function gatherResponse(response) {
  const { headers } = response
  const contentType = headers.get('content-type')
  if (contentType.includes('application/json')) {
    return await response.json()
  } 
  else if (contentType.includes('application/text')) {
    return await response.text()
  } 
  else if (contentType.includes('text/html')) {
    return await response.text()
  } 
  else {
    return await response.text()
  }
}

/**
 * readRequestBody reads in the incoming request body
 * Use await readRequestBody(..) in an async function to get the string
 * @param {Request} request the incoming request to read from
 */
async function readRequestBody(request) {
  const { headers } = request
  const contentType = headers.get('content-type')
  if (contentType.includes('application/json')) {
    const body = await request.json()
    //return JSON.stringify(body)
    return body
  } 
  else if (contentType.includes('application/text')) {
    const body = await request.text()
    return body
  } 
  else if (contentType.includes('text/html')) {
    const body = await request.text()
    return body
  } 
  else if (contentType.includes('form')) {
    const formData = await request.formData()
    let body = {}
    for (let entry of formData.entries()) {
      body[entry[0]] = entry[1]
    }
    return JSON.stringify(body)
  } 
  else {
    let myBlob = await request.blob()
    var objectURL = URL.createObjectURL(myBlob)
    return objectURL
  }
}
