<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gerenciador de Senhas</title>
    <link rel="manifest" href="manifest.json">
    <script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-database.js"></script>
    <script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-auth.js"></script>
    <style>
        body { font-family: Arial, sans-serif; text-align: center; }
        button { font-size: 20px; padding: 10px; margin: 10px; }
        #senha-atual { font-size: 30px; font-weight: bold; }
        #customizacao { margin: 20px; }
    </style>
</head>
<body>
    <h1 id="titulo">Gerenciador de Senhas</h1>
    <div id="customizacao">
        <label>Cor de fundo: <input type="color" id="corFundo" onchange="customizar()"></label>
        <label>Nome do painel: <input type="text" id="nomePainel" onchange="customizar()"></label>
    </div>
    <div id="login">
        <h2>Login</h2>
        <input type="email" id="email" placeholder="Email">
        <input type="password" id="senha" placeholder="Senha">
        <button onclick="login()">Entrar</button>
    </div>
    <div id="painel" style="display: none;">
        <button onclick="gerarSenha()">Gerar Senha</button>
        <h2>Senha Atual: <span id="senha-atual">Nenhuma</span></h2>
        <button onclick="chamarSenha()">Chamar Senha</button>
        <ul id="fila-senhas"></ul>
    </div>
    
    <script>
        if ('serviceWorker' in navigator) {
            navigator.serviceWorker.register('/service-worker.js')
                .then(() => console.log('Service Worker registrado!'))
                .catch(err => console.error('Erro ao registrar o Service Worker', err));
        }

        const firebaseConfig = {
            apiKey: "SUA_API_KEY",
            authDomain: "SEU_DOMINIO.firebaseapp.com",
            databaseURL: "https://SEU_DATABASE.firebaseio.com",
            projectId: "SEU_PROJECT_ID",
            storageBucket: "SEU_BUCKET.appspot.com",
            messagingSenderId: "SEU_SENDER_ID",
            appId: "SEU_APP_ID"
        };
        firebase.initializeApp(firebaseConfig);
        const db = firebase.database();
        const auth = firebase.auth();

        function login() {
            const email = document.getElementById("email").value;
            const senha = document.getElementById("senha").value;
            auth.signInWithEmailAndPassword(email, senha)
                .then(() => {
                    document.getElementById("login").style.display = "none";
                    document.getElementById("painel").style.display = "block";
                })
                .catch(error => alert("Erro ao fazer login: " + error.message));
        }

        function gerarSenha() {
            const novaSenha = "S" + Math.floor(Math.random() * 1000);
            db.ref("senhas").push(novaSenha);
        }

        function chamarSenha() {
            db.ref("senhas").once("value", snapshot => {
                const senhas = snapshot.val();
                if (senhas) {
                    const primeiraChave = Object.keys(senhas)[0];
                    const senhaAtual = senhas[primeiraChave];
                    document.getElementById("senha-atual").innerText = senhaAtual;
                    db.ref("senhas/" + primeiraChave).remove();
                }
            });
        }

        function customizar() {
            document.body.style.backgroundColor = document.getElementById("corFundo").value;
            document.getElementById("titulo").innerText = document.getElementById("nomePainel").value;
        }
    </script>
</body>
</html>
