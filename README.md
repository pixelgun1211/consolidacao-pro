<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Controle de Filas</title>
  <script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-auth-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-firestore-compat.js"></script>
  <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-gray-950 text-gray-100 min-h-screen p-6">

  <div class="max-w-6xl mx-auto space-y-10">
    <header class="text-center">
      <h1 class="text-3xl font-bold">📋 Controle de Filas</h1>
    </header>

    <!-- Formulário de Entrada -->
    <section class="bg-gray-900 p-6 rounded-lg shadow-lg border border-gray-700">
      <h2 class="text-xl font-semibold mb-4">Registrar Motorista</h2>
      <form id="entryForm" class="space-y-4">
        <input type="text" id="nome" placeholder="Nome do Motorista" class="w-full p-2 rounded bg-gray-800 border border-gray-600">
        <input type="text" id="placa" placeholder="Placa do Caminhão" class="w-full p-2 rounded bg-gray-800 border border-gray-600">
        <select id="transportadora" class="w-full p-2 rounded bg-gray-800 border border-gray-600">
          <option value="">Selecione a Transportadora</option>
          <option value="1">Transportadora A</option>
          <option value="2">Transportadora B</option>
          <option value="3">Transportadora C</option>
          <option value="4">Transportadora D</option>
        </select>
        <button type="submit" class="w-full bg-indigo-600 hover:bg-indigo-700 text-white py-2 rounded">Adicionar à Fila</button>
      </form>
    </section>

    <!-- Painel das Filas -->
    <section>
      <h2 class="text-xl font-semibold mb-4">Filas</h2>
      <div id="queuesContainer" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-2 gap-6"></div>
    </section>
  </div>

  <!-- Toast -->
  <div id="toast" class="hidden fixed bottom-5 right-5 bg-gray-800 text-white px-4 py-2 rounded shadow-lg"></div>

  <script type="module">
    // =================== Firebase ===================
    import { initializeApp } from "https://www.gstatic.com/firebasejs/9.22.2/firebase-app.js";
    import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/9.22.2/firebase-auth.js";
    import { getFirestore, collection, addDoc, onSnapshot, query, orderBy } from "https://www.gstatic.com/firebasejs/9.22.2/firebase-firestore.js";

    const firebaseConfig = {
      apiKey: "SUA_API_KEY",
      authDomain: "SEU_PROJETO.firebaseapp.com",
      projectId: "SEU_PROJETO",
      storageBucket: "SEU_PROJETO.appspot.com",
      messagingSenderId: "SENDER_ID",
      appId: "APP_ID"
    };

    const app = initializeApp(firebaseConfig);
    const auth = getAuth(app);
    const db = getFirestore(app);

    // =================== Estado ===================
    const state = {
      transportadoras: [
        { id: "1", nome: "Transportadora A" },
        { id: "2", nome: "Transportadora B" },
        { id: "3", nome: "Transportadora C" },
        { id: "4", nome: "Transportadora D" }
      ],
      db: db
    };

    // =================== UI ===================
    const ui = {
      entryForm: document.getElementById("entryForm"),
      queuesContainer: document.getElementById("queuesContainer"),
      showToast: (msg, type = "info") => {
        const toast = document.getElementById("toast");
        toast.textContent = msg;
        toast.className = `fixed bottom-5 right-5 px-4 py-2 rounded shadow-lg text-white ${type === "success" ? "bg-green-600" : type === "error" ? "bg-red-600" : "bg-gray-800"}`;
        toast.classList.remove("hidden");
        setTimeout(() => toast.classList.add("hidden"), 3000);
      },
      renderQueueContainers: () => {
        ui.queuesContainer.innerHTML = "";
        state.transportadoras.forEach(t => {
          const queueEl = document.createElement("div");
          queueEl.id = `queue-${t.id}`;
          queueEl.className = "p-6 bg-gray-900 rounded-lg shadow-lg border border-gray-700 flex flex-col h-full";
          queueEl.innerHTML = `
            <h2 class="text-xl font-semibold mb-4">${t.nome}</h2>
            <ul id="list-${t.id}" class="space-y-2"></ul>
          `;
          ui.queuesContainer.appendChild(queueEl);
        });
      }
    };

    // =================== Services ===================
    const services = {
      addEntry: async (nome, placa, transportadora) => {
        const collectionPath = `filas/${transportadora}/entradas`;
        const snapshot = await onSnapshot(query(collection(db, collectionPath), orderBy("createdAt", "desc")));
        let nextQueueNumber = 1;
        snapshot.forEach(doc => {
          nextQueueNumber = doc.data().senha + 1;
          return;
        });

        await addDoc(collection(state.db, collectionPath), {
          nome,
          placa,
          senha: nextQueueNumber,
          createdAt: new Date()
        });
        return nextQueueNumber;
      },
      setupFirestoreListener: () => {
        state.transportadoras.forEach(t => {
          const q = query(collection(db, `filas/${t.id}/entradas`), orderBy("createdAt", "asc"));
          onSnapshot(q, (snapshot) => {
            const listEl = document.getElementById(`list-${t.id}`);
            listEl.innerHTML = "";
            snapshot.forEach(doc => {
              const li = document.createElement("li");
              const d = doc.data();
              li.textContent = `${d.senha} - ${d.nome} (${d.placa})`;
              listEl.appendChild(li);
            });
          });
        });
      }
    };

    // =================== Inicialização ===================
    ui.renderQueueContainers();

    onAuthStateChanged(auth, (user) => {
      if (user) {
        services.setupFirestoreListener();

        // 🔑 só agora ativamos o form
        ui.entryForm.addEventListener("submit", async (e) => {
          e.preventDefault();
          const nome = document.getElementById("nome").value.trim();
          const placa = document.getElementById("placa").value.trim().toUpperCase();
          const transportadora = document.getElementById("transportadora").value;

          try {
            const senha = await services.addEntry(nome, placa, transportadora);
            ui.showToast(`Senha gerada: ${senha}`, "success");
            ui.entryForm.reset();
          } catch (error) {
            ui.showToast("Erro ao registrar entrada", "error");
            console.error("Erro ao adicionar entrada:", error);
          }
        });

      } else {
        const token = typeof __initial_auth_token !== "undefined" ? __initial_auth_token : null;
        const authPromise = token ? signInWithCustomToken(auth, token) : signInAnonymously(auth);
        authPromise.catch(error => {
          ui.showToast("Erro de autenticação", "error");
          console.error("Erro de autenticação:", error);
        });
      }
    });
  </script>
</body>
</html>
