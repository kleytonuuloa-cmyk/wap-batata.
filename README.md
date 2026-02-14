<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Wap Batata</title>
    <script src="https://unpkg.com/html5-qrcode"></script>
    <script src="https://www.gstatic.com/firebasejs/9.17.1/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.17.1/firebase-database-compat.js"></script>
    <style>
        /* O DESIGN QUE VOCÊ GOSTA (Igual às fotos do Netlify) */
        * { margin: 0; padding: 0; box-sizing: border-box; font-family: 'Arial', sans-serif; }
        body { background-color: #fff9e6; height: 100vh; overflow: hidden; }

        /* Login Estilizado */
        #login { position: fixed; inset: 0; background: #d32f2f; display: flex; justify-content: center; align-items: center; z-index: 999; }
        .login-box { background: white; padding: 30px; border-radius: 20px; text-align: center; width: 90%; max-width: 320px; box-shadow: 0 10px 30px rgba(0,0,0,0.2); }
        .login-box h2 { color: #d32f2f; margin-bottom: 20px; }
        .login-box input { width: 100%; padding: 12px; margin: 8px 0; border: 2px solid #ffcc00; border-radius: 10px; }
        .login-box button { width: 100%; padding: 12px; background: #d32f2f; color: white; border: none; border-radius: 10px; font-weight: bold; cursor: pointer; }

        /* Menu Lateral Arredondado das Fotos */
        #sistema { display: none; height: 100vh; }
        .sidebar { width: 160px; background: #d32f2f; padding: 15px; display: flex; flex-direction: column; gap: 10px; border-radius: 0 30px 30px 0; }
        .sidebar h2 { color: white; font-size: 18px; margin-bottom: 10px; }
        
        .btn-menu { 
            background: #ffcc00; color: #b71c1c; border: none; 
            padding: 12px; border-radius: 12px; font-weight: bold; 
            font-size: 13px; cursor: pointer; text-align: center;
            box-shadow: 0 4px 0 #e6b800; transition: 0.2s;
        }
        .btn-menu:active { transform: translateY(3px); box-shadow: none; }

        .container { flex: 1; padding: 15px; overflow-y: auto; display: flex; flex-direction: column; }
        
        /* Cards do Estilo Original */
        .card { 
            background: white; padding: 20px; border-radius: 20px; 
            box-shadow: 0 5px 15px rgba(0,0,0,0.08); margin-bottom: 15px;
            border-left: 8px solid #d32f2f;
        }
        h3 { color: #333; margin-bottom: 12px; font-size: 18px; }

        input, select { width: 100%; padding: 12px; margin-bottom: 10px; border: 1px solid #ddd; border-radius: 10px; background: #fafafa; }
        
        .btn-principal { background: #d32f2f; color: white; border: none; padding: 15px; border-radius: 12px; width: 100%; font-weight: bold; cursor: pointer; }
        .btn-verde { background: #27ae60; margin-bottom: 10px; }

        /* Chat e Cozinha */
        #chatBox { height: 250px; overflow-y: auto; background: #fdfdfd; border-radius: 10px; padding: 10px; border: 1px solid #eee; margin-bottom: 10px; display: flex; flex-direction: column; gap: 8px; }
        .msg { padding: 8px 12px; border-radius: 12px; max-width: 85%; font-size: 14px; }
        .msg.me { align-self: flex-end; background: #d32f2f; color: white; }
        .msg.other { align-self: flex-start; background: #eee; color: #333; }

        table { width: 100%; border-collapse: collapse; margin-top: 10px; }
        th { text-align: left; color: #666; font-size: 12px; padding-bottom: 8px; }
        td { padding: 10px 0; border-top: 1px solid #eee; font-size: 14px; color: #444; }

        .page { display: none; }
    </style>
</head>
<body>

    <div id="login">
        <div class="login-box">
            <h2>🌭 Wap Batata</h2>
            <input type="text" id="user" placeholder="Seu nome">
            <input type="password" id="pass" placeholder="Senha">
            <button onclick="login()">ENTRAR</button>
        </div>
    </div>

    <div id="sistema" style="display: flex;">
        <div class="sidebar">
            <h2>Wap Batata</h2>
            <button class="btn-menu" onclick="ver('venda')">💰 VENDER</button>
            <button class="btn-menu" onclick="ver('estoque')">📦 ESTOQUE</button>
            <button class="btn-menu" onclick="ver('cozinha')">👨‍🍳 COZINHA</button>
            <button class="btn-menu" onclick="ver('chat')">💬 CHAT</button>
            <button class="btn-menu" onclick="ver('scanner')">📸 SCANNER</button>
            <button class="btn-menu" style="background:white; color:red; margin-top:auto" onclick="location.reload()">SAIR</button>
        </div>

        <div class="container">
            <div id="venda" class="page">
                <div class="card">
                    <h3>Novo Pedido</h3>
                    <select id="selectProd"></select>
                    <input type="number" id="qtdVenda" value="1" min="1">
                    <button class="btn-principal btn-verde" onclick="addCarrinho()">+ Adicionar ao Pedido</button>
                </div>
                <div class="card">
                    <ul id="listaVenda" style="list-style:none; margin-bottom:10px;"></ul>
                    <h3 style="color:#27ae60">Total: R$ <span id="totalVenda">0.00</span></h3>
                    <input type="number" id="vrecebido" placeholder="Valor Recebido" oninput="calcTroco()">
                    <p><strong>Troco: R$ <span id="troco">0.00</span></strong></p>
                    <button class="btn-principal" onclick="finalizar()">FECHAR E ENVIAR</button>
                </div>
            </div>

            <div id="estoque" class="page">
                <div class="card">
                    <h3>Cadastrar Batata/Item</h3>
                    <input type="text" id="c_barras" placeholder="Código (Bipe aqui)">
                    <input type="text" id="c_nome" placeholder="Nome da Batata">
                    <input type="number" id="c_preco" placeholder="Preço R$">
                    <input type="number" id="c_qtd" placeholder="Qtd em Estoque">
                    <button class="btn-principal" onclick="salvar()">SALVAR NO BANCO</button>
                </div>
                <div class="card">
                    <table>
                        <thead><tr><th>Item</th><th>Qtd</th><th>Ação</th></tr></thead>
                        <tbody id="tabelaEstoque"></tbody>
                    </table>
                </div>
            </div>

            <div id="cozinha" class="page">
                <div class="card">
                    <h3>Pedidos na Cozinha</h3>
                    <div id="pedidosCozinha"></div>
                </div>
            </div>

            <div id="chat" class="page">
                <div class="card">
                    <h3>Chat Interno</h3>
                    <div id="chatBox"></div>
                    <div style="display:flex; gap:5px">
                        <input type="text" id="msgInput" placeholder="Sua mensagem...">
                        <button onclick="mandarMsg()" style="padding:10px; border-radius:10px; background:#d32f2f; color:white; border:none;">OK</button>
                    </div>
                </div>
            </div>

            <div id="scanner" class="page">
                <div class="card">
                    <div id="reader" style="width:100%; border-radius:15px; overflow:hidden;"></div>
                </div>
            </div>
        </div>
    </div>

<script>
// CONFIGURAÇÃO DO SEU FIREBASE
const firebaseConfig = {
  apiKey: "AIzaSyAonNwRaURIP8sBYnE9Au9DT2jhcBzjQnk",
  authDomain: "wap-batata-recheada.firebaseapp.com",
  projectId: "wap-batata-recheada",
  databaseURL: "https://wap-batata-recheada-default-rtdb.firebaseio.com/", // Link direto do seu banco
  storageBucket: "wap-batata-recheada.firebasestorage.app",
  messagingSenderId: "960647817419",
  appId: "1:960647817419:web:0af3204cc1a972eae4d34e"
};

firebase.initializeApp(firebaseConfig);
const db = firebase.database();
let user = "";
let cart = [];
let total = 0;
let scanner;

function login() {
    user = document.getElementById("user").value.toLowerCase();
    let p = document.getElementById("pass").value;
    if(["talyta","kleyton","rafael","kemilly"].includes(user) && p === "2312") {
        document.getElementById("login").style.display="none";
        ver('venda');
        rodarBanco();
    } else { alert("Acesso Negado!"); }
}

function ver(id) {
    document.querySelectorAll('.page').forEach(p => p.style.display="none");
    document.getElementById(id).style.display="block";
    if(id==='scanner') startScan(); else stopScan();
}

function rodarBanco() {
    // Sincronizar Estoque
    db.ref('estoque').on('value', snap => {
        let data = snap.val();
        let tab = document.getElementById("tabelaEstoque");
        let sel = document.getElementById("selectProd");
        tab.innerHTML = ""; sel.innerHTML = "";
        for(let id in data) {
            tab.innerHTML += `<tr><td>${data[id].nome}</td><td>${data[id].qtd}</td><td><button onclick="db.ref('estoque/${id}').remove()">X</button></td></tr>`;
            sel.innerHTML += `<option value="${id}">${data[id].nome} (R$ ${data[id].preco})</option>`;
        }
    });

    // Sincronizar Cozinha
    db.ref('pedidos').on('value', snap => {
        let peds = snap.val();
        let box = document.getElementById("pedidosCozinha");
        box.innerHTML = "";
        for(let id in peds) {
            box.innerHTML += `<div class="card" style="border-left-color:#27ae60"><strong>${peds[id].user}:</strong><br>${peds[id].msg}<br><button class="btn-principal" onclick="db.ref('pedidos/${id}').remove()" style="background:#27ae60; margin-top:10px">CONCLUÍDO</button></div>`;
        }
    });

    // Sincronizar Chat
    db.ref('chat').on('value', snap => {
        let msgs = snap.val();
        let box = document.getElementById("chatBox");
        box.innerHTML = "";
        for(let id in msgs) {
            let classe = msgs[id].user === user ? 'me' : 'other';
            box.innerHTML += `<div class="msg ${classe}"><strong>${msgs[id].user}:</strong> ${msgs[id].texto}</div>`;
        }
        box.scrollTop = box.scrollHeight;
    });
}

function addCarrinho() {
    let id = document.getElementById("selectProd").value;
    let q = parseInt(document.getElementById("qtdVenda").value);
    db.ref('estoque/'+id).once('value').then(s => {
        let p = s.val();
        cart.push({id, nome: p.nome, preco: p.preco, qtd: q});
        total += (p.preco * q);
        document.getElementById("totalVenda").innerText = total.toFixed(2);
        document.getElementById("listaVenda").innerHTML += `<li>${p.nome} x${q}</li>`;
    });
}

function calcTroco() {
    let rec = parseFloat(document.getElementById("vrecebido").value) || 0;
    document.getElementById("troco").innerText = (rec - total).toFixed(2);
}

function finalizar() {
    if(cart.length === 0) return;
    let resumo = "";
    cart.forEach(i => {
        db.ref('estoque/'+i.id).transaction(d => { if(d) d.qtd -= i.qtd; return d; });
        resumo += i.nome + " x" + i.qtd + " | ";
    });
    db.ref('pedidos').push({ msg: resumo, user: user });
    alert("Pedido enviado!");
    cart = []; total = 0;
    document.getElementById("listaVenda").innerHTML = "";
    document.getElementById("totalVenda").innerText = "0.00";
}

function salvar() {
    let id = document.getElementById("c_barras").value || document.getElementById("c_nome").value;
    db.ref('estoque/'+id).set({
        nome: document.getElementById("c_nome").value,
        preco: parseFloat(document.getElementById("c_preco").value),
        qtd: parseInt(document.getElementById("c_qtd").value)
    });
    alert("Sincronizado!");
}

function mandarMsg() {
    let t = document.getElementById("msgInput").value;
    if(t) db.ref('chat').push({user: user, texto: t});
    document.getElementById("msgInput").value = "";
}

function startScan() {
    scanner = new Html5Qrcode("reader");
    scanner.start({ facingMode: "environment" }, { fps: 10, qrbox: 250 }, 
    res => { document.getElementById("c_barras").value = res; document.getElementById("selectProd").value = res; ver('venda'); });
}
function stopScan() { if(scanner) scanner.stop().then(()=>scanner=null); }
</script>
</body>
</html>
