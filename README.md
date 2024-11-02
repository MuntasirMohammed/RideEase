<p align="left">
<img src="https://imgur.com/rBaBLAA.png" height="10%" width="10%" alt="RideEase"/>
<h1>RideEase</h1>


<h2>Description</h2>
RideEase is a serverless ride-sharing application built to demonstrate proficiency in AWS cloud services and full-stack development. This app integrates a variety of AWS technologies, including Amplify (Gen 2) for streamlined deployment, Cognito for user authentication, Lambda for backend logic, and API Gateway for secure API management. Using DynamoDB for scalable data storage and GitHub for version control, RideEase delivers a robust experience from user login to real-time ride requests. This project showcases an in-depth understanding of serverless architecture, authentication, and data management in a fully functional, scalable web app..
<br />

<h2>Live Application</h2>
🔗 <a href="https://rideease.engmuntasir.com/" target="_blank">Check out the Live version here!</a>

<h2>Features</h2>

- <b>User Authentication:</b> Integrated using AWS Cognito for secure access.
- <b>CI/CD Pipeline:</b> Set up via GitHub for automated builds and deployment.
- <b>Live Hosting:</b> Deployed on AWS Amplify, accessible through a public URL.
- <b>Database Management:</b> DynamoDB used for efficient data storage and retrieval.
- <b>Backend Logic:</b> Lambda functions to handle serverless backend processing.
- <b>API Management:</b> API Gateway for secure and scalable API endpoint management.

<h2>AWS Services Used</h2>

- <b>GitHub:</b> Version control and CI/CD integration.
- <b>Amplify:</b> Hosting and deployment of the React app.
- <b>Cognito:</b> User authentication and management.
- <b>DynamoDB:</b> Data storage solution.
- <b>Lambda:</b> Serverless functions for backend logic.
- <b>API Gateway:</b> Managing and securing API endpoints.
- <b>IAM:</b> Managing access permissions.

<h2>Application Walk-Through :</h2>
<p align="center">
Application Architecture: <br/>
<img src="https://imgur.com/03pENQz.png" height="60%" width="60%" alt="Application Architecture"/>
<br />
<p align="center">
<br> Register New Account <br/>
<img src="https://imgur.com/cDY7R8O.png" height="70%" width="70%" alt="Create New Account"/>
<br />
<br> Map <br/>
<img src="https://imgur.com/4DrcmJV.png" height="70%" width="70%" alt="map"/>
<br> Ride Updates <br/>
<img src="https://imgur.com/ugYDnW0.png" height="40%" width="35%" alt="ride progress"/>
<br> Ride details updated in Dynamo DB <br/>
<img src="https://imgur.com/1NXjkFY.png" height="68%" width='68%' alt="table"/>

<h2>Lambda Function Code Used:</h2>

```node
import { randomBytes } from 'crypto';
import { DynamoDBClient } from '@aws-sdk/client-dynamodb';
import { DynamoDBDocumentClient, PutCommand } from '@aws-sdk/lib-dynamodb';

const client = new DynamoDBClient({});
const ddb = DynamoDBDocumentClient.from(client);

const fleet = [
    { Name: 'Alice', Color: 'White Prius', Gender: 'Female' },
    { Name: 'BOB', Color: 'White Accord', Gender: 'Male' },
    { Name: 'Charlie', Color: 'Yellow Mustang', Gender: 'Female' },
];

export const handler = async (event, context) => {
    if (!event.requestContext.authorizer) {
        return errorResponse('Authorization not configured', context.awsRequestId);
    }

    const rideId = toUrlString(randomBytes(16));
    console.log('Received event (', rideId, '): ', event);

    const username = event.requestContext.authorizer.claims['cognito:username'];
    const requestBody = JSON.parse(event.body);
    const pickupLocation = requestBody.PickupLocation;

    const unicorn = findUnicorn(pickupLocation);

    try {
        await recordRide(rideId, username, unicorn);
        return {
            statusCode: 201,
            body: JSON.stringify({
                RideId: rideId,
                Unicorn: unicorn,
                Eta: '30 seconds',
                Rider: username,
            }),
            headers: {
                'Access-Control-Allow-Origin': '*',
            },
        };
    } catch (err) {
        console.error(err);
        return errorResponse(err.message, context.awsRequestId);
    }
};

function findUnicorn(pickupLocation) {
    console.log('Finding Driver for ', pickupLocation.Latitude, ', ', pickupLocation.Longitude);
    return fleet[Math.floor(Math.random() * fleet.length)];
}

async function recordRide(rideId, username, unicorn) {
    const params = {
        TableName: 'Rides',
        Item: {
            RideID: rideId,
            User: username,
            Unicorn: unicorn,
            RequestTime: new Date().toISOString(),
        },
    };
    await ddb.send(new PutCommand(params));
}

function toUrlString(buffer) {
    return buffer.toString('base64')
        .replace(/\+/g, '-')
        .replace(/\//g, '_')
        .replace(/=/g, '');
}

function errorResponse(errorMessage, awsRequestId) {
    return {
        statusCode: 500,
        body: JSON.stringify({
            Error: errorMessage,
            Reference: awsRequestId,
        }),
        headers: {
            'Access-Control-Allow-Origin': '*',
        },
    };
}
```

