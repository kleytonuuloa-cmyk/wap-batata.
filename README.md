<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Wap Batata - Sistema Completo</title>
    <script src="https://unpkg.com/html5-qrcode"></script>
    <script src="https://www.gstatic.com/firebasejs/9.17.1/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.17.1/firebase-database-compat.js"></script>
    <style>
        *{margin:0;padding:0;box-sizing:border-box;font-family:Arial;} 
        body{height:100vh;overflow:hidden;background:#fff8e1} 
        #login{height:100vh;display:flex;justify-content:center;align-items:center;background:linear-gradient(45deg,#ffcc00,#ff0000);background-image:url('https://images.unsplash.com/photo-1617196031636-c5d45b4a1f47?fit=crop&w=1400&q=80');background-size:cover;background-position:center;}
        .loginBox{background:white;padding:30px;border-radius:15px;width:90%;max-width:350px;text-align:center;box-shadow:0 0 20px rgba(0,0,0,0.3);}
        .loginBox h2{color:#d32f2f;margin-bottom:15px;}
        .loginBox input{width:100%;padding:12px;margin:10px 0;border:2px solid #ffcc00;border-radius:8px;}
        .loginBox button{width:100%;padding:12px;background:#d32f2f;color:white;border:none;border-radius:8px;cursor:pointer;font-weight:bold;}
        #sistema{display:none;height:100vh;display:flex;}
        .sidebar{width:140px;background:#d32f2f;color:white;padding:10px;display:flex;flex-direction:column;font-size:10px;}
        .sidebar button{background:#ffcc00;border:none;color:#b71c1c;padding:10px;margin-bottom:5px;cursor:pointer;font-weight:bold;border-radius:6px;font-size:10px;}
        .content{flex:1;padding:15px;overflow:auto;background:#fffde7;}
        .pagina{display:none;}
        .card{background:white;padding:12px;border-radius:12px;margin-bottom:10px;box-shadow:0 3px 10px rgba(0,0,0,0.1);border-left:5px solid #d32f2f;}
        table{width:100%;border-collapse:collapse;margin-top:5px;color:black;font-size:11px;}
        th{background:#d32f2f;color:white;padding:5px;}
        td{border:1px solid #ccc;padding:5px;text-align:center;}
        .action{background:#d32f2f;color:white;border:none;padding:10px;cursor:pointer;border-radius:6px;font-weight:bold;width:100%;margin-top:5px;}
        .formInput{width:100%;padding:10px;margin-top:5px;border:1px solid #ccc;border-radius:6px;}
        #chatBox{height:200px;overflow-y:auto;border:1px solid #ddd;padding:10px;margin-bottom:10px;background:#f9f9f9;font-size:12px;display:flex;flex-direction:column;}
        .msg{margin-bottom:8px;padding:5px 10px;border-radius:10px;max-width:80%;}
        .msg.me{background:#dcf8c6;align-self:flex-end;}
        .msg.other{background:#eee;align-self:flex-start;}
    </style>
</head>
<body>

<div id="login">
    <div class="loginBox">
        <h2>🌭 Wap Batata</h2>
        <input type="text" id="user" placeholder="Usuário">
        <input type="password" id="pass" placeholder="Senha">
        <button onclick="login()">Entrar</button>
    </div>
</div>

<div id="sistema">
    <div class="sidebar">
        <h4 id="boasVindas">Wap Batata</h4>
        <button onclick="mostrar('pedidos')">💰 VENDER</button>
        <button onclick="mostrar('cardapio')">📦 ESTOQUE</button>
        <button onclick="mostrar('cozinha')">👨‍🍳 COZINHA</button>
        <button onclick="mostrar('chat')">💬 CHAT</button>
        <button onclick="mostrar('barcode')">📸 SCANNER</button>
        <button onclick="location.reload()" style="margin-top:auto;background:white;color:red">Sair</button>
    </div>

    <div class="content">
        <div id="pedidos" class="pagina">
            <div class="card">
                <h3>Venda</h3>
                <select id="selectProd" class="formInput"></select>
                <input type="number" id="qtdVenda" value="1" min="1" class="formInput">
                <button class="action" style="background:#27ae60" onclick="adicionarAoCarrinho()">🛒 Adicionar</button>
            </div>
            <div class="card" style="color:black">
                <ul id="carrinho"></ul>
                <hr>
                <h3>Total: R$ <span id="valorTotal">0.00</span></h3>
                <input type="number" id="valorPago" class="formInput" placeholder="Valor pago pelo cliente" oninput="calcularTroco()">
                <h4 style="color:green">Troco: R$ <span id="valorTroco">0.00</span></h4>
                <button class="action" style="background:#d35400" onclick="finalizarVenda()">Finalizar e Enviar Cozinha</button>
            </div>
        </div>

        <div id="cardapio" class="pagina">
            <div class="card">
                <h3>Cadastrar Produto</h3>
                <input type="text" id="codItem" class="formInput" placeholder="Bip do Scanner">
                <input type="text" id="nomeItem" class="formInput" placeholder="Nome">
                <input type="number" id="precoItem" class="formInput" placeholder="Preço R$">
                <input type="number" id="estoqueItem" class="formInput" placeholder="Qtd em Estoque">
                <button class="action" onclick="salvarItem()">Sincronizar</button>
            </div>
            <div class="card">
                <table>
                    <thead><tr><th>Item</th><th>R$</th><th>Qtd</th><th>X</th></tr></thead>
                    <tbody id="corpoEstoque"></tbody>
                </table>
            </div>
        </div>

        <div id="cozinha" class="pagina">
            <div class="card"><h3>Pedidos na Fila</h3><ul id="listaCozinha" style="color:black"></ul></div>
        </div>

        <div id="chat" class="pagina">
            <div class="card">
                <h3>Chat da Equipe</h3>
                <div id="chatBox"></div>
                <input type="text" id="msgInput" class="formInput" placeholder="Digite uma mensagem...">
                <button class="action" onclick="enviarMensagem()">Enviar</button>
            </div>
        </div>

        <div id="barcode" class="pagina">
            <div class="card">
                <div id="barcodeScanner" style="width:100%; min-height:250px; background:#000;"></div>
                <button class="action" onclick="mostrar('pedidos')">Voltar</button>
            </div>
        </div>
    </div>
</div>

<script>
const firebaseConfig = {
  apiKey: "AIzaSyAonNwRaURIP8sBYnE9Au9DT2jhcBzjQnk",
  authDomain: "wap-batata-recheada.firebaseapp.com",
  projectId: "wap-batata-recheada",
  storageBucket: "wap-batata-recheada.firebasestorage.app",
  messagingSenderId: "960647817419",
  appId: "1:960647817419:web:0af3204cc1a972eae4d34e",
  measurementId: "G-KX3W7K5C1F"
};

firebase.initializeApp(firebaseConfig);
const db = firebase.database();
let usuarioAtual = "";
let carrinho = [];
let totalVenda = 0;
let html5QrCode;

function login(){
    usuarioAtual = document.getElementById("user").value.toLowerCase();
    let p = document.getElementById("pass").value;
    if(["talyta","kleyton","rafael","kemilly"].includes(usuarioAtual) && p === "2312"){
        document.getElementById("login").style.display="none";
        document.getElementById("sistema").style.display="flex";
        document.getElementById("boasVindas").innerText = "Olá, " + usuarioAtual;
        ouvirBanco();
    } else { alert("Login inválido!"); }
}

function mostrar(id){
    document.querySelectorAll(".pagina").forEach(p=>p.style.display="none");
    document.getElementById(id).style.display="block";
    if(id === 'barcode') startScanner(); else stopScanner();
}

function calcularTroco(){
    let pago = parseFloat(document.getElementById("valorPago").value) || 0;
    let troco = pago - totalVenda;
    document.getElementById("valorTroco").innerText = troco > 0 ? troco.toFixed(2) : "0.00";
}

function salvarItem(){
    let cod = document.getElementById("codItem").value || document.getElementById("nomeItem").value;
    let nome = document.getElementById("nomeItem").value;
    let preco = parseFloat(document.getElementById("precoItem").value);
    let qtd = parseInt(document.getElementById("estoqueItem").value) || 0;
    if(!nome || !preco) return;
    db.ref('estoque/' + cod).set({ nome, preco, qtd });
    alert("Salvo!");
}

function ouvirBanco(){
    db.ref('estoque').on('value', (snapshot) => {
        let data = snapshot.val();
        let corpo = document.getElementById("corpoEstoque");
        let select = document.getElementById("selectProd");
        corpo.innerHTML = ""; select.innerHTML = "";
        for(let id in data){
            let item = data[id];
            let row = corpo.insertRow();
            row.insertCell(0).innerText = item.nome;
            row.insertCell(1).innerText = item.preco;
            row.insertCell(2).innerText = item.qtd;
            row.insertCell(3).innerHTML = `<button onclick="db.ref('estoque/${id}').remove()">X</button>`;
            let opt = document.createElement("option");
            opt.value = id; opt.text = item.nome + " (Qtd: "+item.qtd+")";
            select.appendChild(opt);
        }
    });

    db.ref('pedidos').on('value', (snapshot) => {
        let peds = snapshot.val();
        let lista = document.getElementById("listaCozinha");
        lista.innerHTML = "";
        for(let id in peds){
            let li = document.createElement("li");
            li.style.padding="10px"; li.style.borderBottom="1px solid #ddd";
            li.innerHTML = `<strong>${peds[id].msg}</strong><br><small>${peds[id].hora}</small> <button onclick="db.ref('pedidos/${id}').remove()" style="float:right">FEITO</button>`;
            lista.appendChild(li);
        }
    });

    db.ref('chat').on('value', (snapshot) => {
        let msgs = snapshot.val();
        let box = document.getElementById("chatBox");
        box.innerHTML = "";
        for(let id in msgs){
            let div = document.createElement("div");
            div.className = "msg " + (msgs[id].user === usuarioAtual ? "me" : "other");
            div.innerHTML = `<strong>${msgs[id].user}:</strong> ${msgs[id].texto}`;
            box.appendChild(div);
        }
        box.scrollTop = box.scrollHeight;
    });
}

function adicionarAoCarrinho(){
    let id = document.getElementById("selectProd").value;
    let qtd = parseInt(document.getElementById("qtdVenda").value);
    db.ref('estoque/' + id).once('value').then(snap => {
        let p = snap.val();
        if(p.qtd < qtd) return alert("Sem estoque!");
        carrinho.push({id, qtd, nome: p.nome, preco: p.preco});
        totalVenda += (p.preco * qtd);
        document.getElementById("valorTotal").innerText = totalVenda.toFixed(2);
        let li = document.createElement("li");
        li.innerText = p.nome + " x" + qtd;
        document.getElementById("carrinho").appendChild(li);
        calcularTroco();
    });
}

function finalizarVenda(){
    if(carrinho.length === 0) return;
    let resumo = "";
    carrinho.forEach(item => {
        db.ref('estoque/' + item.id).transaction(p => { if(p) p.qtd -= item.qtd; return p; });
        resumo += item.nome + " x" + item.qtd + " ";
    });
    db.ref('pedidos').push({ msg: resumo, hora: new Date().toLocaleTimeString(), user: usuarioAtual });
    alert("Venda enviada!");
    carrinho = []; totalVenda = 0;
    document.getElementById("carrinho").innerHTML = "";
    document.getElementById("valorTotal").innerText = "0.00";
    document.getElementById("valorPago").value = "";
    document.getElementById("valorTroco").innerText = "0.00";
}

function enviarMensagem(){
    let texto = document.getElementById("msgInput").value;
    if(!texto) return;
    db.ref('chat').push({ user: usuarioAtual, texto: texto });
    document.getElementById("msgInput").value = "";
}

function startScanner(){
    html5QrCode = new Html5Qrcode("barcodeScanner");
    html5QrCode.start({ facingMode: "environment" }, { fps:10, qrbox:200 },
        codigo => {
            document.getElementById("selectProd").value = codigo;
            document.getElementById("codItem").value = codigo;
            alert("Bip!"); mostrar('pedidos');
        }
    );
}
function stopScanner(){ if(html5QrCode) html5QrCode.stop().then(()=>html5QrCode=null); }
</script>
</body>
</html>
