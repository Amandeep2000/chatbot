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




app.listen(port, () => {
  console.log(`server is running at ${serverurl}`);

});
