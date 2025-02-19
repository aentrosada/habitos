<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Rastreador de Hábitos</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #f7f7f7;
            color: #333;
        }

        header {
            background-color: #9b4dca;
            color: white;
            text-align: center;
            padding: 1.5rem 0;
            border-bottom: 5px solid #7f3ba6;
        }

        h1 {
            font-size: 2.5rem;
            font-weight: 600;
        }

        p {
            font-size: 1.1rem;
        }

        .container {
            max-width: 1100px;
            margin: 20px auto;
            padding: 20px;
            background: white;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
            border-radius: 12px;
        }

        .login-form, .habit-form {
            display: flex;
            flex-direction: column;
            gap: 12px;
            align-items: center;
        }

        input, button {
            padding: 12px;
            border-radius: 8px;
            border: 1px solid #ddd;
            font-size: 1rem;
            width: 80%; /* Ajustado para ocupar menos espaço em telas grandes */
        }

        button {
            background-color: #9b4dca;
            color: white;
            border: none;
            cursor: pointer;
            transition: background-color 0.3s ease;
        }

        button:hover {
            background-color: #7f3ba6;
        }

        .habit-list {
            list-style-type: none;
            padding: 0;
            margin-top: 30px;
        }

        .habit-item {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 12px;
            margin: 8px 0;
            background-color: #fff;
            border: 1px solid #ddd;
            border-radius: 8px;
            box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);
        }

        .habit-item span {
            flex: 1;
            font-size: 1.1rem;
        }

        .habit-item button {
            background-color: #9b4dca;
            color: white;
            padding: 6px 12px;
            border-radius: 8px;
            border: none;
            cursor: pointer;
            transition: background-color 0.3s ease;
        }

        .habit-item button:hover {
            background-color: #7f3ba6;
        }

        .delete-button {
            background-color: #e040fb;
            color: white;
            padding: 6px 12px;
            border-radius: 8px;
            border: none;
            cursor: pointer;
            transition: background-color 0.3s ease;
        }

        .delete-button:hover {
            background-color: #d500f9;
        }

        .medal {
            font-weight: bold;
            color: #FFD700;
        }

        .hidden {
            display: none;
        }

        /* Responsividade */
        @media (max-width: 768px) {
            header {
                padding: 1rem;
            }

            h1 {
                font-size: 2rem;
            }

            .container {
                margin: 10px;
                padding: 15px;
            }

            input, button {
                padding: 10px;
                font-size: 0.95rem;
                width: 100%; /* Agora os campos ocuparão 100% da largura em dispositivos pequenos */
            }

            .habit-item {
                padding: 10px;
                flex-direction: column;
                align-items: flex-start;
            }

            .habit-item button {
                width: 100%;
                margin-top: 8px;
            }

            .login-form input, .login-form button {
                width: 100%; /* Assegura que os campos e botões de login/cadastro ocupem 100% da largura em dispositivos móveis */
            }
        }
    </style>
</head>
<body>
    <header>
        <h1>Rastreador de Hábitos</h1>
        <p>Acompanhe seus hábitos e mantenha a consistência!</p>
    </header>

    <div class="container">
        <!-- Login Section -->
        <form class="login-form" id="login-form">
            <input type="email" id="email" placeholder="Digite seu email" required>
            <input type="password" id="password" placeholder="Digite sua senha" required>
            <button type="submit">Entrar</button>
            <button type="button" id="register-button">Cadastrar</button>
        </form>

        <!-- Habit Tracker Section -->
        <div id="habit-section" class="hidden">
            <form class="habit-form" id="habit-form">
                <input type="text" id="habit-name" placeholder="Adicione um novo hábito" required>
                <button type="submit">Adicionar Hábito</button>
            </form>
            <ul class="habit-list" id="habit-list"></ul>
            <button id="logout-button">Sair</button>
        </div>
    </div>

    <script>
        const loginForm = document.getElementById('login-form');
        const habitSection = document.getElementById('habit-section');
        const habitForm = document.getElementById('habit-form');
        const habitNameInput = document.getElementById('habit-name');
        const habitList = document.getElementById('habit-list');
        const registerButton = document.getElementById('register-button');
        const logoutButton = document.getElementById('logout-button');

        let currentUser = null;
        let users = JSON.parse(localStorage.getItem('users')) || {};

        function getMedal(streak) {
            if (streak >= 90) return '🏆 Platina';
            if (streak >= 75) return '💎 Diamante';
            if (streak >= 60) return '🥇 Ouro III';
            if (streak >= 45) return '🥇 Ouro II';
            if (streak >= 30) return '🥇 Ouro I';
            if (streak >= 21) return '🥈 Prata';
            if (streak >= 15) return '🥉 Bronze I';
            if (streak >= 11) return '🥉 Bronze';
            if (streak >= 3) return '⭐ BETA';
            return '';
        }

        function renderHabits() {
            habitList.innerHTML = '';
            const userHabits = users[currentUser].habits || [];

            userHabits.forEach((habit, index) => {
                const habitItem = document.createElement('li');
                habitItem.className = 'habit-item';

                const habitName = document.createElement('span');
                const medal = getMedal(habit.streak);
                habitName.innerHTML = `${habit.name} - Sequência: ${habit.streak} dias <span class="medal">${medal}</span>`;

                const completeButton = document.createElement('button');
                completeButton.textContent = 'Marcar como Feito';
                completeButton.onclick = () => {
                    habit.streak++;
                    saveUsers();
                    renderHabits();
                };

                const resetButton = document.createElement('button');
                resetButton.textContent = 'Reiniciar Sequência';
                resetButton.style.backgroundColor = '#f44336';
                resetButton.onclick = () => {
                    habit.streak = 0;
                    saveUsers();
                    renderHabits();
                };

                const deleteButton = document.createElement('button');
                deleteButton.className = 'delete-button';
                deleteButton.textContent = 'Excluir';
                deleteButton.onclick = () => {
                    users[currentUser].habits.splice(index, 1);
                    saveUsers();
                    renderHabits();
                };

                habitItem.appendChild(habitName);
                habitItem.appendChild(completeButton);
                habitItem.appendChild(resetButton);
                habitItem.appendChild(deleteButton);

                habitList.appendChild(habitItem);
            });
        }

        function saveUsers() {
            localStorage.setItem('users', JSON.stringify(users));
        }

        loginForm.addEventListener('submit', (e) => {
            e.preventDefault();
            const email = document.getElementById('email').value;
            const password = document.getElementById('password').value;

            if (users[email] && users[email].password === password) {
                currentUser = email;
                loginForm.classList.add('hidden');
                habitSection.classList.remove('hidden');
                renderHabits();
            } else {
                alert('Credenciais inválidas ou usuário não encontrado.');
            }
        });

        registerButton.addEventListener('click', () => {
            const email = document.getElementById('email').value;
            const password = document.getElementById('password').value;

            if (email && password) {
                if (!users[email]) {
                    users[email] = { password, habits: [] };
                    saveUsers();
                    alert('Cadastro realizado com sucesso! Faça login para continuar.');
                } else {
                    alert('Usuário já cadastrado.');
                }
            } else {
                alert('Por favor, preencha email e senha para cadastrar.');
            }
        });

        habitForm.addEventListener('submit', (e) => {
            e.preventDefault();
            const habitName = habitNameInput.value.trim();
            if (habitName) {
                users[currentUser].habits.push({ name: habitName, streak: 0 });
                habitNameInput.value = '';
                saveUsers();
                renderHabits();
            }
        });

        logoutButton.addEventListener('click', () => {
            currentUser = null;
            habitSection.classList.add('hidden');
            loginForm.classList.remove('hidden');
        });
    </script>
</body>
</html>


