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
        :root { --primary: #d32f2f; --secondary: #ffcc00; --bg: #f4f4f4; }
        * { margin: 0; padding: 0; box-sizing: border-box; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; }
        body { background: var(--bg); height: 100vh; display: flex; flex-direction: column; overflow: hidden; }
        
        /* Login Profissional */
        #login { position: fixed; inset: 0; z-index: 1000; background: linear-gradient(135deg, var(--primary), #8e0000); display: flex; justify-content: center; align-items: center; padding: 20px; }
        .login-card { background: white; padding: 40px; border-radius: 20px; width: 100%; max-width: 400px; text-align: center; box-shadow: 0 10px 25px rgba(0,0,0,0.3); }
        .login-card h2 { color: var(--primary); margin-bottom: 25px; font-size: 28px; }
        .login-card input { width: 100%; padding: 15px; margin-bottom: 15px; border: 2px solid #ddd; border-radius: 10px; outline: none; transition: 0.3s; }
        .login-card input:focus { border-color: var(--secondary); }
        .login-card button { width: 100%; padding: 15px; background: var(--primary); color: white; border: none; border-radius: 10px; font-weight: bold; font-size: 16px; cursor: pointer; }

        /* Estrutura Principal */
        #sistema { display: none; height: 100vh; width: 100%; }
        .sidebar { width: 100px; background: var(--primary); display: flex; flex-direction: column; align-items: center; padding-top: 20px; box-shadow: 2px 0 10px rgba(0,0,0,0.1); }
        .nav-item { width: 70px; height: 70px; background: var(--secondary); border-radius: 15px; margin-bottom: 15px; display: flex; flex-direction: column; justify-content: center; align-items: center; cursor: pointer; border: none; font-size: 10px; font-weight: bold; color: #b71c1c; transition: 0.2s; }
        .nav-item:active { transform: scale(0.9); }
        .nav-item i { font-size: 20px; margin-bottom: 5px; }

        .main-content { flex: 1; padding: 20px; overflow-y: auto; }
        .page { display: none; animation: fadeIn 0.3s ease; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }

        /* Cards e Estilo */
        .card { background: white; padding: 20px; border-radius: 15px; box-shadow: 0 4px 6px rgba(0,0,0,0.05); margin-bottom: 20px; }
        h3 { margin-bottom: 15px; color: #333; font-size: 18px; border-left: 4px solid var(--primary); padding-left: 10px; }
        .input-group { margin-bottom: 15px; }
        .input-group label { display: block; margin-bottom: 5px; font-size: 14px; font-weight: bold; color: #666; }
        input, select { width: 100%; padding: 12px; border: 1px solid #ddd; border-radius: 8px; font-size: 14px; }
        
        .btn-action { width: 100%; padding: 15px; background: var(--primary); color: white; border: none; border-radius: 8px; font-weight: bold; cursor: pointer; margin-top: 10px; }
        .btn-success { background: #2e7d32; }

        /* Tabela de Estoque */
        table { width: 100%; border-collapse: collapse; }
        th { text-align: left; padding: 12px; background: #f8f8f8; color: #666; font-size: 13px; }
        td { padding: 12px; border-top: 1px solid #eee; font-size: 14px; }
        .badge { padding: 4px 8px; border-radius: 4px; font-size: 12px; font-weight: bold; }
        .badge-stock { background: #e8f5e9; color: #2e7d32; }

        /* Chat Estilizado */
        #chatBox { height: 300px; overflow-y: auto; display: flex; flex-direction: column; gap: 10px; padding: 10px; background: #f9f9f9; border-radius: 10px; border: 1px solid #eee; margin-bottom: 10px; }
        .msg { padding: 10px 15px; border-radius: 15px; max-width: 80%; font-size: 14px; line-height: 1.4; }
        .msg.me { align-self: flex-end; background: var(--primary); color: white; border-bottom-right-radius: 2px; }
        .msg.other { align-self: flex-start; background: #eee; color: #333; border-bottom-left-radius: 2px; }
    </style>
</head>
<body>

    <div id="login">
        <div class="login-card">
            <h2>Wap Batata</h2>
            <input type="text" id="user" placeholder="Seu nome">
            <input type="password" id="pass" placeholder="Senha">
            <button onclick="login()">ACESSAR SISTEMA</button>
        </div>
    </div>

    <div id="sistema">
        <div class="sidebar">
            <button class="nav-item" onclick="mostrar('venda')">💰<br>VENDER</button>
            <button class="nav-item" onclick="mostrar('estoque')">📦<br>ESTOQUE</button>
            <button class="nav-item" onclick="mostrar('cozinha')">👨‍🍳<br>COZINHA</button>
            <button class="nav-item" onclick="mostrar('chat')">💬<br>CHAT</button>
            <button class="nav-item" onclick="mostrar('scanner')">📸<br>SCAN</button>
            <button class="nav-item" style="background:#fff; color:red; margin-top:auto" onclick="location.reload()">SAIR</button>
        </div>

        <div class="main-content">
            <div id="venda" class="page">
                <div class="card">
                    <h3>Realizar Venda</h3>
                    <div class="input-group">
                        <label>Produto</label>
                        <select id="selectProd"></select>
                    </div>
                    <div class="input-group">
                        <label>Quantidade</label>
                        <input type="number" id="qtdVenda" value="1" min="1">
                    </div>
                    <button class="btn-action btn-success" onclick="adicionarAoCarrinho()">ADICIONAR ITEM</button>
                </div>
                <div class="card">
                    <ul id="carrinho" style="margin-bottom: 15px; list-style: none;"></ul>
                    <div style="font-size: 20px; font-weight: bold; margin-bottom: 10px;">Total: R$ <span id="valorTotal">0.00</span></div>
                    <input type="number" id="valorPago" placeholder="Valor recebido" style="margin-bottom: 10px;" oninput="calcularTroco()">
                    <div style="color: #2e7d32; font-weight: bold;">Troco: R$ <span id="valorTroco">0.00</span></div>
                    <button class="btn-action" onclick="finalizarVenda()">FECHAR PEDIDO</button>
                </div>
            </div>

            <div id="estoque" class="page">
                <div class="card">
                    <h3>Novo Item</h3>
                    <input type="text" id="codItem" placeholder="Código de barras" class="input-group">
                    <input type="text" id="nomeItem" placeholder="Nome do produto" class="input-group">
                    <input type="number" id="precoItem" placeholder="Preço (ex: 25.00)" class="input-group">
                    <input type="number" id="estoqueItem" placeholder="Quantidade inicial" class="input-group">
                    <button class="btn-action" onclick="salvarItem()">SALVAR NO SISTEMA</button>
                </div>
                <div class="card">
                    <table>
                        <thead><tr><th>Produto</th><th>Qtd</th><th>Ação</th></tr></thead>
                        <tbody id="corpoEstoque"></tbody>
                    </table>
                </div>
            </div>

            <div id="cozinha" class="page"><div class="card"><h3>Pedidos Pendentes</h3><div id="listaCozinha"></div></div></div>
            <div id="chat" class="page">
                <div class="card">
                    <h3>Chat da Equipe</h3>
                    <div id="chatBox"></div>
                    <div style="display:flex; gap: 10px;">
                        <input type="text" id="msgInput" placeholder="Sua mensagem...">
                        <button onclick="enviarMensagem()" style="padding:10px; background:var(--primary); color:white; border:none; border-radius:8px;">OK</button>
                    </div>
                </div>
            </div>
            
            <div id="scanner" class="page"><div class="card"><h3>Scanner</h3><div id="barcodeScanner" style="width:100%; border-radius:10px; overflow:hidden;"></div></div></div>
        </div>
    </div>

<script>
// CONFIGURAÇÃO DO FIREBASE (ORIGINAL)
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

function login() {
    usuarioAtual = document.getElementById("user").value.toLowerCase();
    let p = document.getElementById("pass").value;
    if(["talyta","kleyton","rafael","kemilly"].includes(usuarioAtual) && p === "2312"){
        document.getElementById("login").style.display="none";
        document.getElementById("sistema").style.display="flex";
        mostrar('venda');
        ouvirBanco();
    } else { alert("Usuário ou senha incorretos."); }
}

function mostrar(id) {
    document.querySelectorAll(".page").forEach(p => p.style.display="none");
    document.getElementById(id).style.display="block";
    if(id === 'scanner') startScanner(); else stopScanner();
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
    if(!nome || !preco) return alert("Preencha Nome e Preço!");
    db.ref('estoque/' + cod).set({ nome, preco, qtd });
    alert("Item Sincronizado!");
    document.getElementById("nomeItem").value = ""; document.getElementById("precoItem").value = "";
}

function ouvirBanco(){
    db.ref('estoque').on('value', (snap) => {
        let data = snap.val();
        let corpo = document.getElementById("corpoEstoque");
        let select = document.getElementById("selectProd");
        corpo.innerHTML = ""; select.innerHTML = "";
        for(let id in data){
            let row = corpo.insertRow();
            row.innerHTML = `<td>${data[id].nome}</td><td><span class="badge badge-stock">${data[id].qtd}</span></td><td><button onclick="db.ref('estoque/${id}').remove()">Excluir</button></td>`;
            let opt = document.createElement("option");
            opt.value = id; opt.text = `${data[id].nome} (R$ ${data[id].preco})`;
            select.appendChild(opt);
        }
    });

    db.ref('pedidos').on('value', (snap) => {
        let peds = snap.val();
        let lista = document.getElementById("listaCozinha");
        lista.innerHTML = "";
        for(let id in peds){
            let div = document.createElement("div");
            div.className = "card";
            div.innerHTML = `<strong>Pedido de: ${peds[id].user}</strong><p>${peds[id].msg}</p><button class="btn-action" onclick="db.ref('pedidos/${id}').remove()">CONCLUÍDO</button>`;
            lista.appendChild(div);
        }
    });

    db.ref('chat').on('value', (snap) => {
        let msgs = snap.val();
        let box = document.getElementById("chatBox");
        box.innerHTML = "";
        for(let id in msgs){
            let div = document.createElement("div");
            div.className = "msg " + (msgs[id].user === usuarioAtual ? "me" : "other");
            div.innerHTML = `<strong>${msgs[id].user}:</strong><br>${msgs[id].texto}`;
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
        li.innerText = `${p.nome} x${qtd}`;
        document.getElementById("carrinho").appendChild(li);
        calcularTroco();
    });
}

function finalizarVenda(){
    if(carrinho.length === 0) return;
    let resumo = "";
    carrinho.forEach(item => {
        db.ref('estoque/' + item.id).transaction(p => { if(p) p.qtd -= item.qtd; return p; });
        resumo += `${item.nome} (x${item.qtd}) `;
    });
    db.ref('pedidos').push({ msg: resumo, hora: new Date().toLocaleTimeString(), user: usuarioAtual });
    alert("Venda Finalizada!");
    carrinho = []; totalVenda = 0;
    document.getElementById("carrinho").innerHTML = "";
    document.getElementById("valorTotal").innerText = "0.00";
    document.getElementById("valorPago").value = "";
}

function enviarMensagem(){
    let texto = document.getElementById("msgInput").value;
    if(!texto) return;
    db.ref('chat').push({ user: usuarioAtual, texto: texto });
    document.getElementById("msgInput").value = "";
}

function startScanner(){
    html5QrCode = new Html5Qrcode("barcodeScanner");
    html5QrCode.start({ facingMode: "environment" }, { fps:10, qrbox:250 },
        codigo => { document.getElementById("selectProd").value = codigo; mostrar('venda'); }
    );
}
function stopScanner(){ if(html5QrCode) html5QrCode.stop().then(()=>html5QrCode=null); }
</script>
</body>
</html>

