# CodeClause_project_BuildaBasicChatApplicationUsingWebSockets

<!--javascript code-->
import { createServer } from 'http';
import staticHandler from 'serve-handler';
import ws, { WebSocketServer } from 'ws';
//serve static folder
const server = createServer((req, res) => {   // (1)
    return staticHandler(req, res, { public: 'public' })
});
const wss = new WebSocketServer({ server }) // (2)
wss.on('connection', (client) => {
    console.log('Client connected !')
    client.on('message', (msg) => {    // (3)
        console.log(`Message:${msg}`);
        broadcast(msg)
    })
})
function broadcast(msg) {       // (4)
    for (const client of wss.clients) {
        if (client.readyState === ws.OPEN) {
            client.send(msg)
        }
    }
}
server.listen(process.argv[2] || 8080, () => {
    console.log(`server listening...`);
})

<!--html code-->
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="index.css">
    <title>Minimlist chat App</title>
</head>

<body>
    <div class="container">
        <p class="msg">Messages:</p>
        <div id="messages" class="messages"></div>
        <form id="msgForm" class="msgForm">
            <input type="text" placeholder="Send message" class="input" id="inputBox" />
            <input type="submit" class="btn" value="Send">
        </form>
    </div>
    <script type="text/javascript">
        const ws = new WebSocket(`ws://${window.document.location.host}`);
        ws.binaryType = "blob";
        // Log socket opening and closing
        ws.addEventListener("open", event => {
            console.log("Websocket connection opened");
        });
        ws.addEventListener("close", event => {
            console.log("Websocket connection closed");
        });
        ws.onmessage = function (message) {
            const msgDiv = document.createElement('div');
            msgDiv.classList.add('msgCtn');
            if (message.data instanceof Blob) {
                reader = new FileReader();
                reader.onload = () => {
                    msgDiv.innerHTML = reader.result;
                    document.getElementById('messages').appendChild(msgDiv);
                };
                reader.readAsText(message.data);
            } else {
                console.log("Result2: " + message.data);
                msgDiv.innerHTML = message.data;
                document.getElementById('messages').appendChild(msgDiv);
            }
        }
        const form = document.getElementById('msgForm');
        form.addEventListener('submit', (event) => {
            event.preventDefault();
            const message = document.getElementById('inputBox').value;
            ws.send(message);
            document.getElementById('inputBox').value = ''
        })
    </script>
</body>

</html>

<!--css code-->
html{
font-size: 62.5%;
}
body{
margin:0;
padding: 0;
display:flex;
justify-content:center;
}
.container{
margin-top: 5rem;
display:flex;
flex-direction:column;
width: 30%;
height: 70vh;
}
.msg{
color:blueviolet;
font-size: 2rem;
}
.msgCtn{
margin: 1rem 0;
color: white;
padding: 1rem 1rem;
background-color: blueviolet;
border-radius: 6px;
font-size: 2rem
}
.msgForm{
display:flex;
flex-direction:row;
}
.input{
height: 100%;
flex: 4;
margin-right: 2rem;
border:none;
border-radius: .5rem;
padding: 2px 1rem;
font-size: 1.5rem;
background-color:aliceblue;
}
.input:focus{
background-color:white;
}
.btn{
height:5rem;
flex: 1;
display:flex;
justify-content:center;
align-items:center;
border-radius: .5rem;
background-color:blueviolet;
color: whitesmoke;
border:none;
font-size: 1.6rem;
}
