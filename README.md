# chatbot
const express = require("express");
const port = 8000;
const serverurl = `http://localhost:${port}`;
const app = express();
const {google}=require("googleapis")
// const promptes= require("./src/routers/router");
const { calendar } = require("googleapis/build/src/apis/calendar");
const oauthclient= new google.auth.OAuth2(process.env.ClientID,process.env.Redirect,process.env.SecretID)
// app.use(express.json());
// app.use(promptes)


app.get('/', (req, res) => {
  const url = oauthclient.generateAuthUrl({
    access_type: 'offline',
    scope: 'https://www.googleapis.com/auth/calendar.readonly'
  });
  res.redirect(url);
});

app.get('/redirect',(req,res)=>{
  const code=req.query.code;
  oauthclient.getToken(code,(err,tokens)=>{
    if(err){
      console.error('could not  get token',err);
      res.send('Error');
      return
    }
    oauthclient.setCredentials(tokens);
  return  res.send('SucessFully logeed in');
  })
  
})


app.get('/calender',(req,res)=>{
  const canlender=google.calendar({version:'v3',auth:oauthclient});
  canlender.calendarList.list({},(err,response)=>{
    if(err){
      console.error('error fetching calender',err)
      res.end('Error');
      // res.json
      return
    }
    const calendar=response.data.items
    res.json(calendar)
  })
})


app.get('/event', (req, res) => {
  const calendarId = req.query.calendar || 'primary';
  const calendar = google.calendar({ version: 'v3', auth: oauthclient });
  calendar.events.list({
    calendarId,
    timeMin: (new Date()).toISOString(),
    maxResults: 15,
    singleEvents: true,
    orderBy: 'startTime'
  }, (err, response) => {
    if (err) {
      console.log('Can\'t fetch events:', err);
      res.send('Error');
      return;
    }
    const events = response.data.items;
    res.json(events);
  });
});


//event create
const express = require('express');
const { google } = require('googleapis');
const bodyParser = require('body-parser');
const fs = require('fs');
const path = require('path');

const app = express();
const PORT = process.env.PORT || 3000;

app.use(bodyParser.json());

const SCOPES = ['https://www.googleapis.com/auth/calendar'];
const TOKEN_PATH = path.join(__dirname, 'token.json');
const CREDENTIALS_PATH = path.join(__dirname, 'credentials.json');

let oAuth2Client;

fs.readFile(CREDENTIALS_PATH, (err, content) => {
  if (err) return console.log('Error loading client secret file:', err);
  authorize(JSON.parse(content));
});

function authorize(credentials) {
  const { client_secret, client_id, redirect_uris } = credentials.installed;
  oAuth2Client = new google.auth.OAuth2(client_id, client_secret, redirect_uris[0]);

  fs.readFile(TOKEN_PATH, (err, token) => {
    if (err) return getAccessToken();
    oAuth2Client.setCredentials(JSON.parse(token));
  });
}

function getAccessToken() {
  const authUrl = oAuth2Client.generateAuthUrl({
    access_type: 'offline',
    scope: SCOPES,
  });
  console.log('Authorize this app by visiting this url:', authUrl);
}

app.post('/book-appointment', async (req, res) => {
  const { summary, description, start, end } = req.body;

  const calendar = google.calendar({ version: 'v3', auth: oAuth2Client });

  const event = {
    summary,
    description,
    start: {
      dateTime: start,
      timeZone: 'America/Los_Angeles',
    },
    end: {
      dateTime: end,
      timeZone: 'America/Los_Angeles',
    },
  };

  try {
    const response = await calendar.events.insert({
      calendarId: 'primary',
      resource: event,
    });
    res.status(200).send(`Event created: ${response.data.htmlLink}`);
  } catch (error) {
    res.status(500).send(`Error creating event: ${error}`);
  }
});

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});



app.listen(port, () => {
  console.log(`server is running at ${serverurl}`);

});
