<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Batalha Naval</title>
    <!-- Chosen Palette: Serene Earth Tones -->
    <!-- Application Structure Plan: Uma SPA com layout responsivo de três colunas para desktop (controles, tabuleiro do jogador, tabuleiro do oponente) e uma coluna empilhada para mobile. Os controles (botões de modo e mensagens) ficam à esquerda. O tabuleiro superior mostra os próprios navios do jogador e os tiros do oponente. O tabuleiro inferior é o tabuleiro do oponente onde o jogador clica para atirar. Esta estrutura foi escolhida para proporcionar uma visão clara e simultânea de ambos os lados do jogo, facilitando a interação (clicar para atirar) e o feedback visual (acertos/erros) para o usuário, promovendo um fluxo de jogo intuitivo. -->
    <!-- Visualization & Content Choices: Tabuleiros de Jogo (Objetivo: Representar o estado da matriz 10x10 para cada jogador; Método: Divs HTML com CSS Grid; Interação: Células clicáveis no tabuleiro do oponente para "atirar"; Justificativa: Visualização direta do estado de jogo e interação primária. Símbolos de Estado (Objetivo: Identificação clara de água, navio, acerto, erro, habilidade; Método: Números/caracteres textuais (0, 3, X, O, 5); Justificativa: Leve, claro e alinhado com a simplicidade numérica do PDF. Mensagens de Jogo (Objetivo: Informar turno, acertos, erros, afundamentos, vitória; Método: Painel de texto dedicado; Interação: Atualização dinâmica via JS; Justificativa: Feedback crucial em tempo real. Botões de Modo (Objetivo: Alternar entre o jogo e demonstrações de habilidades; Método: Botões HTML; Interação: Clique para mudar o modo; Justificativa: Permite explorar diferentes aspectos do desafio do PDF (jogo vs. habilidades especiais/posicionamento). -->
    <!-- CONFIRMATION: NO SVG graphics used. NO Mermaid JS used. -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body {
            font-family: 'Inter', sans-serif;
            margin: 0;
            padding: 0;
            background-color: #F7F5EE; /* Warm neutral background */
            color: #333;
        }
        .container {
            max-width: 1200px;
            margin: 0 auto;
            padding: 1.5rem;
        }
        .game-grid {
            display: grid;
            grid-template-columns: 1fr; /* Default para mobile */
            gap: 2rem;
        }
        @media (min-width: 768px) {
            .game-grid {
                grid-template-columns: 1fr 1fr 1fr; /* Três colunas para desktop */
            }
        }

        .board-container {
            border: 2px solid #5C5B5A; /* Darker border for board */
            border-radius: 0.5rem;
            overflow: hidden;
            background-color: #E6F3F9; /* Light blue for water background */
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
        }

        .board-grid-inner {
            display: grid;
            grid-template-columns: repeat(10, 2.2rem); /* 10 cols */
            grid-template-rows: repeat(10, 2.2rem);    /* 10 rows */
            width: fit-content;
            margin: 0 auto;
        }

        .board-cell {
            width: 2.2rem;
            height: 2.2rem;
            display: flex;
            justify-content: center;
            align-items: center;
            font-weight: bold;
            font-size: 0.9rem;
            border: 1px solid rgba(0,0,0,0.08); /* Subtle cell borders */
            transition: background-color 0.15s ease-in-out;
        }

        /* Responsive cell sizing */
        @media (max-width: 768px) {
            .board-grid-inner {
                grid-template-columns: repeat(10, 1.8rem);
                grid-template-rows: repeat(10, 1.8rem);
            }
            .board-cell {
                width: 1.8rem;
                height: 1.8rem;
                font-size: 0.8rem;
            }
        }
        
        /* Cell States (valores do PDF: 0=água, 3=navio, 5=habilidade) */
        .val-0 { background-color: #E6F3F9; color: #333; } /* Água */
        .val-1 { background-color: #7A5D45; color: #FFF; } /* Navio (representação interna) */
        .val-3 { background-color: #7A5D45; color: #FFF; } /* Navio (representação PDF) */
        .val-5 { background-color: #FFC107; color: #FFF; } /* Habilidade (representação PDF) */
        .val-X { background-color: #E74C3C; color: #FFF; }  /* Acerto */
        .val-O { background-color: #3498DB; color: #FFF; } /* Erro */

        .clickable-cell { cursor: pointer; }
        .clickable-cell:hover { background-color: rgba(0,0,0,0.1); } /* Hover effect for clickable cells */

        .section-header {
            font-size: 1.25rem;
            font-weight: bold;
            color: #555;
            margin-bottom: 0.75rem;
            text-align: center;
        }

        button {
            transition: all 0.2s ease-in-out;
        }
        button:hover {
            transform: translateY(-2px);
            box-shadow: 0 4px 8px rgba(0,0,0,0.2);
        }
    </style>
</head>
<body class="bg-gray-100 min-h-screen flex flex-col items-center py-8">
    <div class="container bg-white shadow-xl rounded-lg p-6 md:p-8">
        <h1 class="text-4xl font-extrabold text-center mb-6 text-gray-800">Batalha Naval</h1>
        <p class="text-center text-lg text-gray-600 mb-8">Posicione seus navios, lance suas habilidades e afunde a frota inimiga!</p>

        <div class="game-grid">
            <!-- Coluna de Controles e Mensagens -->
            <div class="flex flex-col p-4 bg-gray-50 rounded-lg shadow-inner">
                <h2 class="text-2xl font-semibold text-gray-700 mb-4 text-center">Controles do Jogo</h2>
                <div class="flex-grow">
                    <p id="message-display" class="text-lg font-bold text-gray-800 text-center mb-4 min-h-[3rem]">Bem-vindo à Batalha Naval! Pressione "Iniciar Jogo" para jogar.</p>
                    <div class="space-y-4 mb-6">
                        <button id="start-game-button" class="w-full bg-blue-600 hover:bg-blue-700 text-white py-3 px-4 rounded-xl font-bold">
                            Iniciar Jogo
                        </button>
                        <button id="reset-game-button" class="w-full bg-red-600 hover:bg-red-700 text-white py-3 px-4 rounded-xl font-bold">
                            Reiniciar Jogo
                        </button>
                    </div>
                </div>
                <!-- Demonstrações dos Módulos do PDF -->
                <div class="mt-auto pt-4 border-t border-gray-200">
                    <h3 class="text-xl font-semibold text-gray-700 mb-3 text-center">Modos de Demonstração</h3>
                    <p class="text-sm text-gray-600 mb-3 text-center">Visualize o posicionamento e as habilidades.</p>
                    <div class="grid grid-cols-1 gap-3">
                        <button id="novato-demo-button" class="bg-gray-400 hover:bg-gray-500 text-white py-2 px-3 rounded-lg font-semibold text-sm">Módulo Novato (Navios H/V)</button>
                        <button id="aventureiro-demo-button" class="bg-gray-400 hover:bg-gray-500 text-white py-2 px-3 rounded-lg font-semibold text-sm">Módulo Aventureiro (Navios Diagonais)</button>
                        <button id="mestre-demo-button" class="bg-gray-400 hover:bg-gray-500 text-white py-2 px-3 rounded-lg font-semibold text-sm">Módulo Mestre (Habilidades)</button>
                    </div>
                </div>
            </div>

            <!-- Tabuleiro do Jogador 1 (Seu Tabuleiro) -->
            <div class="board-container p-4">
                <div class="section-header">Seu Tabuleiro</div>
                <div id="player1-board-display" class="board-grid-inner"></div>
            </div>

            <!-- Tabuleiro do Jogador 2 (Tabuleiro do Oponente) -->
            <div class="board-container p-4">
                <div class="section-header">Tabuleiro do Oponente</div>
                <div id="player2-board-display" class="board-grid-inner"></div>
            </div>
        </div>
    </div>

    <script>
        // --- Constantes do Jogo ---
        const BOARD_SIZE = 10; // Tamanho do tabuleiro (10x10)
        const WATER = 0;       // Representa uma casa de água (Valor PDF: 0)
        const SHIP = 1;        // Representa uma parte de navio (Valor PDF: 3)
        const HIT = 'X';       // Representa um acerto em navio
        const MISS = 'O';      // Representa um tiro na água
        const ABILITY_EFFECT = 5; // Representa uma casa afetada por habilidade (Valor PDF: 5)

        // Mapeamento de valores internos para exibição conforme o PDF
        const DISPLAY_VALUES = {
            [WATER]: '0',
            [SHIP]: '3',
            [HIT]: 'X',
            [MISS]: 'O',
            [ABILITY_EFFECT]: '5'
        };

        // Definições dos navios: ID, tamanho e número de acertos
        // Mantido o array original para o jogo jogável, com comprimentos variados
        const SHIP_TYPES = [
            { id: 'carrier', length: 5, hits: 0, sunk: false, positions: [] },
            { id: 'battleship', length: 4, hits: 0, sunk: false, positions: [] },
            { id: 'destroyer1', length: 3, hits: 0, sunk: false, positions: [] },
            { id: 'destroyer2', length: 3, hits: 0, sunk: false, positions: [] },
            { id: 'submarine', length: 2, hits: 0, sunk: false, positions: [] }
        ];

        // --- Variáveis de Estado do Jogo ---
        let player1Board; // Tabuleiro do Jogador 1 (mostra seus navios e tiros do oponente)
        let player2Board; // Tabuleiro do Jogador 2 (mostra seus navios e tiros do oponente)
        let player1Ships; // Estado dos navios do Jogador 1
        let player2Ships; // Estado dos navios do Jogador 2
        let currentPlayer; // 'player1' ou 'player2'
        let gameOver = false;
        let demoMode = false; // Flag para indicar se estamos em modo de demonstração

        // --- Referências aos Elementos do DOM ---
        const messageDisplay = document.getElementById('message-display');
        const player1BoardDisplay = document.getElementById('player1-board-display');
        const player2BoardDisplay = document.getElementById('player2-board-display');
        const startGameButton = document.getElementById('start-game-button');
        const resetGameButton = document.getElementById('reset-game-button');
        const novatoDemoButton = document.getElementById('novato-demo-button');
        const aventureiroDemoButton = document.getElementById('aventureiro-demo-button');
        const mestreDemoButton = document.getElementById('mestre-demo-button');

        // --- Funções de Ajuda ---

        /**
         * @function initializeBoardData
         * @brief Cria uma nova matriz de tabuleiro preenchida com água (valor WATER).
         * @returns {Array<Array<number>>} Uma nova matriz 10x10 de água.
         */
        function initializeBoardData() {
            return Array(BOARD_SIZE).fill(0).map(() => Array(BOARD_SIZE).fill(WATER));
        }

        /**
         * @function resetShipData
         * @brief Reinicializa o array de tipos de navios para um novo jogo.
         * Retorna uma cópia limpa dos tipos de navios com hits e posições resetados.
         * @returns {Array<Object>} Uma cópia limpa dos tipos de navios.
         */
        function resetShipData() {
            return SHIP_TYPES.map(type => ({
                id: type.id,
                length: type.length,
                hits: 0,
                sunk: false,
                positions: []
            }));
        }

        /**
         * @function updateMessage
         * @brief Atualiza a mensagem exibida na interface do usuário.
         * @param {string} msg - A mensagem a ser exibida.
         */
        function updateMessage(msg) {
            message = msg;
            messageDisplay.textContent = msg;
        }

        /**
         * @function renderBoard
         * @brief Renderiza um tabuleiro específico no DOM.
         * @param {HTMLElement} displayElement - O elemento DIV onde o tabuleiro será renderizado.
         * @param {Array<Array<number|string>>} boardData - Os dados da matriz do tabuleiro.
         * @param {boolean} isPlayerBoard - Indica se é o tabuleiro do jogador (mostra navios) ou do oponente (esconde navios).
         * @param {boolean} isDemo - Indica se está em modo de demonstração (afeta cliques e exibição de todos os valores).
         */
        function renderBoard(displayElement, boardData, isPlayerBoard, isDemo = false) {
            displayElement.innerHTML = ''; // Limpa o tabuleiro atual
            for (let rowIndex = 0; rowIndex < BOARD_SIZE; rowIndex++) {
                for (let colIndex = 0; colIndex < BOARD_SIZE; colIndex++) {
                    const cell = document.createElement('div');
                    cell.classList.add('board-cell');
                    const cellValue = boardData[rowIndex][colIndex];

                    let displayValue = '';
                    let className = '';

                    // Lógica de exibição e classes CSS com base no valor da célula
                    if (cellValue === WATER) {
                        displayValue = DISPLAY_VALUES[WATER];
                        className = 'val-0';
                    } else if (cellValue === SHIP) {
                        if (isPlayerBoard || isDemo) { // No tabuleiro do jogador ou em demo, mostre navios
                            displayValue = DISPLAY_VALUES[SHIP];
                            className = 'val-3'; // Usar 3 para navio conforme PDF
                        } else { // No tabuleiro do oponente (não demo), esconder navios (mostrar água)
                            displayValue = DISPLAY_VALUES[WATER];
                            className = 'val-0';
                        }
                    } else if (cellValue === HIT) {
                        displayValue = DISPLAY_VALUES[HIT];
                        className = 'val-X';
                    } else if (cellValue === MISS) {
                        displayValue = DISPLAY_VALUES[MISS];
                        className = 'val-O';
                    } else if (cellValue === ABILITY_EFFECT) {
                        displayValue = DISPLAY_VALUES[ABILITY_EFFECT];
                        className = 'val-5'; // Usar 5 para habilidade conforme PDF
                    } else { // Fallback, se houver algum valor inesperado, trate como água
                        displayValue = DISPLAY_VALUES[WATER];
                        className = 'val-0';
                    }

                    cell.textContent = displayValue;
                    cell.classList.add(className);

                    // Adiciona o manipulador de clique apenas para o tabuleiro do oponente durante o jogo (não em modo demo)
                    if (!isPlayerBoard && !gameOver && !isDemo) {
                        cell.classList.add('clickable-cell');
                        cell.dataset.row = rowIndex;
                        cell.dataset.col = colIndex;
                        cell.addEventListener('click', handleBoardClick);
                    }
                    displayElement.appendChild(cell);
                }
            }
        }

        /**
         * @function canPlaceShip
         * @brief Verifica se um navio pode ser posicionado no tabuleiro sem sair dos limites ou sobrepor outros navios.
         * @param {number} startRow - Linha inicial.
         * @param {number} startCol - Coluna inicial.
         * @param {number} length - Comprimento do navio.
         * @param {string} orientation - 'horizontal', 'vertical', 'diagonal_down', 'diagonal_up'.
         * @param {Array<Array<number>>} boardToPlaceOn - O tabuleiro onde tentar posicionar.
         * @returns {boolean} True se o navio pode ser posicionado, false caso contrário.
         */
        function canPlaceShip(startRow, startCol, length, orientation, boardToPlaceOn) {
            for (let i = 0; i < length; i++) {
                let r = startRow;
                let c = startCol;

                if (orientation === 'horizontal') {
                    c = startCol + i;
                } else if (orientation === 'vertical') {
                    r = startRow + i;
                } else if (orientation === 'diagonal_down') {
                    r = startRow + i;
                    c = startCol + i;
                } else if (orientation === 'diagonal_up') {
                    r = startRow + i;
                    c = startCol - i;
                }

                if (r < 0 || r >= BOARD_SIZE || c < 0 || c >= BOARD_SIZE || boardToPlaceOn[r][c] === SHIP) {
                    return false;
                }
            }
            return true;
        }

        /**
         * @function placeShip
         * @brief Posiciona um navio no tabuleiro, marcando as casas como SHIP.
         * @param {number} startRow - Linha inicial.
         * @param {number} startCol - Coluna inicial.
         * @param {number} length - Comprimento do navio.
         * @param {string} orientation - 'horizontal', 'vertical', 'diagonal_down', 'diagonal_up'.
         * @param {Array<Array<number>>} boardToPlaceOn - O tabuleiro onde posicionar o navio.
         * @param {Object} [shipObject] - O objeto do navio para atualizar suas posições (opcional, usado no jogo principal).
         * @returns {boolean} True se o navio foi posicionado com sucesso, false caso contrário.
         */
        function placeShip(startRow, startCol, length, orientation, boardToPlaceOn, shipObject) {
            if (!canPlaceShip(startRow, startCol, length, orientation, boardToPlaceOn)) {
                return false;
            }

            if (shipObject) {
                shipObject.positions = [];
            }
            for (let i = 0; i < length; i++) {
                let r = startRow;
                let c = startCol;

                if (orientation === 'horizontal') {
                    c = startCol + i;
                } else if (orientation === 'vertical') {
                    r = startRow + i;
                } else if (orientation === 'diagonal_down') {
                    r = startRow + i;
                    c = startCol + i;
                } else if (orientation === 'diagonal_up') {
                    r = startRow + i;
                    c = startCol - i;
                }
                boardToPlaceOn[r][c] = SHIP;
                if (shipObject) {
                    shipObject.positions.push({ r, c });
                }
            }
            return true;
        }

        /**
         * @function autoPlaceShips
         * @brief Posiciona os navios de forma aleatória no tabuleiro.
         * @param {Array<Array<number>>} boardToPlaceOn - O tabuleiro onde posicionar.
         * @param {Array<Object>} shipsToPlace - A lista de objetos de navios.
         */
        function autoPlaceShips(boardToPlaceOn, shipsToPlace) {
            shipsToPlace.forEach(ship => {
                let placed = false;
                let attempts = 0; // Prevenção de loop infinito em casos muito raros de tabuleiro cheio
                while (!placed && attempts < 1000) {
                    const row = Math.floor(Math.random() * BOARD_SIZE);
                    const col = Math.floor(Math.random() * BOARD_SIZE);
                    const orientationOptions = ['horizontal', 'vertical', 'diagonal_down', 'diagonal_up'];
                    const orientation = orientationOptions[Math.floor(Math.random() * orientationOptions.length)];

                    if (placeShip(row, col, ship.length, orientation, boardToPlaceOn, ship)) {
                        placed = true;
                    }
                    attempts++;
                }
            });
        }

        // --- Lógica do Jogo Principal ---

        /**
         * @function startGame
         * @brief Inicializa e prepara o jogo principal para começar.
         */
        function startGame() {
            demoMode = false;
            player1Board = initializeBoardData();
            player2Board = initializeBoardData();
            player1Ships = resetShipData();
            player2Ships = resetShipData();

            autoPlaceShips(player1Board, player1Ships);
            autoPlaceShips(player2Board, player2Ships);

            currentPlayer = 'player1';
            gameOver = false;
            updateMessage(`Início do Jogo! Turno do Jogador 1. Clique no tabuleiro do oponente para atirar.`);
            renderBoards();
        }

        /**
         * @function handleBoardClick
         * @brief Manipula o clique em uma célula do tabuleiro do oponente (para atirar).
         * @param {Event} event - O evento de clique.
         */
        function handleBoardClick(event) {
            if (gameOver || demoMode) return;

            const row = parseInt(event.target.dataset.row);
            const col = parseInt(event.target.dataset.col);

            const opponentBoardData = (currentPlayer === 'player1') ? player2Board : player1Board;
            const opponentShipsData = (currentPlayer === 'player1') ? player2Ships : player1Ships;

            if (opponentBoardData[row][col] === HIT || opponentBoardData[row][col] === MISS) {
                updateMessage('Você já atirou nesta posição. Escolha outra.');
                return;
            }

            if (opponentBoardData[row][col] === SHIP) {
                opponentBoardData[row][col] = HIT;
                updateMessage(`JOGADOR ${currentPlayer === 'player1' ? 1 : 2} ACERTOU!`);

                let hitShip = null;
                for (const ship of opponentShipsData) {
                    for (const pos of ship.positions) {
                        if (pos.r === row && pos.c === col) {
                            ship.hits++;
                            hitShip = ship;
                            break;
                        }
                    }
                    if (hitShip) break;
                }

                if (hitShip && hitShip.hits === hitShip.length) {
                    hitShip.sunk = true;
                    updateMessage(`JOGADOR ${currentPlayer === 'player1' ? 1 : 2} AFUNDOU UM NAVIO! (${hitShip.id})`);

                    const allSunk = opponentShipsData.every(ship => ship.sunk);
                    if (allSunk) {
                        gameOver = true;
                        updateMessage(`PARABÉNS, JOGADOR ${currentPlayer === 'player1' ? 1 : 2}! VOCÊ AFUNDOU TODOS OS NAVIOS DO OPONENTE!`);
                        renderBoards();
                        return;
                    }
                }
            } else {
                opponentBoardData[row][col] = MISS;
                updateMessage(`JOGADOR ${currentPlayer === 'player1' ? 1 : 2} ERROU!`);
            }

            // Troca o turno
            currentPlayer = (currentPlayer === 'player1') ? 'player2' : 'player1';
            updateMessage(`Turno do Jogador ${currentPlayer === 'player1' ? 1 : 2}.`);
            renderBoards();
        }

        /**
         * @function renderBoards
         * @brief Renderiza ambos os tabuleiros (jogador e oponente) no DOM.
         */
        function renderBoards() {
            renderBoard(player1BoardDisplay, player1Board, true, demoMode);
            renderBoard(player2BoardDisplay, player2Board, false, demoMode);
        }

        /**
         * @function resetGame
         * @brief Reinicia o jogo para o estado inicial, pronto para uma nova partida.
         */
        function resetGame() {
            startGame();
            updateMessage('Jogo reiniciado! Turno do Jogador 1. Clique no tabuleiro do oponente para atirar.');
        }

        // --- Funções de Demonstração dos Módulos do PDF ---

        /**
         * @function runNovatoDemo
         * @brief Demonstra o Módulo Novato: Posicionamento de 2 navios (H e V) de tamanho 3.
         * Alinha-se com o requisito de exibir 0 para água e 3 para navio.
         */
        function runNovatoDemo() {
            demoMode = true;
            const demoBoard = initializeBoardData();

            // Posiciona os navios conforme o desafio do Nível Novato (2 navios de tamanho 3)
            // Navio Horizontal (tamanho 3) em (2,1)
            placeShip(2, 1, 3, 'horizontal', demoBoard, null); // null pois não precisa rastrear hits para demo
            // Navio Vertical (tamanho 3) em (5,5)
            placeShip(5, 5, 3, 'vertical', demoBoard, null);

            updateMessage('Módulo Novato: Tabuleiro com 2 navios (H e V) de tamanho 3. (0 = Água, 3 = Navio)');
            renderBoard(player1BoardDisplay, demoBoard, true, true); // Exibe no tabuleiro do jogador 1
            player2BoardDisplay.innerHTML = ''; // Limpa o tabuleiro do oponente
        }

        /**
         * @function runAventureiroDemo
         * @brief Demonstra o Módulo Aventureiro: Posicionamento de 4 navios (2 H/V, 2 Diagonais) de tamanho 3.
         * Alinha-se com o requisito de exibir 0 para água e 3 para navio.
         */
        function runAventureiroDemo() {
            demoMode = true;
            const demoBoard = initializeBoardData();

            // Posiciona os navios conforme o desafio do Nível Aventureiro (4 navios de tamanho 3)
            // 2 Navios H/V
            placeShip(1, 1, 3, 'horizontal', demoBoard, null);
            placeShip(4, 8, 3, 'vertical', demoBoard, null);
            // 2 Navios Diagonais
            placeShip(0, 0, 3, 'diagonal_down', demoBoard, null); // Diagonal principal
            placeShip(7, 9, 3, 'diagonal_up', demoBoard, null);   // Diagonal secundária

            updateMessage('Módulo Aventureiro: Tabuleiro com 4 navios (H/V e Diagonais) de tamanho 3. (0 = Água, 3 = Navio)');
            renderBoard(player1BoardDisplay, demoBoard, true, true);
            player2BoardDisplay.innerHTML = '';
        }

        /**
         * @function runMestreDemo
         * @brief Demonstra o Módulo Mestre: Áreas de efeito de Habilidades Especiais (Cone, Cruz, Octaedro).
         * Alinha-se com o requisito de exibir 0, 3 e 5.
         */
        function runMestreDemo() {
            demoMode = true;
            const demoBoard = initializeBoardData();

            // Posiciona alguns navios para contexto na demonstração de habilidade
            placeShip(1, 1, 3, 'horizontal', demoBoard, null);
            placeShip(8, 2, 3, 'vertical', demoBoard, null);

            // Cria e aplica as habilidades em pontos fixos para demonstração
            const coneAbility = createConeAbility(7); // Tamanho da matriz de habilidade
            applyAbilityToDemoBoard(coneAbility, 3, 3, demoBoard); // Centro da habilidade no tabuleiro

            const crossAbility = createCrossAbility(5);
            applyAbilityToDemoBoard(crossAbility, 7, 7, demoBoard);

            const octahedronAbility = createOctahedronAbility(5);
            applyAbilityToDemoBoard(octahedronAbility, 2, 8, demoBoard);

            updateMessage('Módulo Mestre: Áreas de Habilidades Especiais (0 = Água, 3 = Navio, 5 = Habilidade).');
            renderBoard(player1BoardDisplay, demoBoard, true, true);
            player2BoardDisplay.innerHTML = '';
        }

        // Funções para criar as matrizes de habilidade (do Módulo Mestre)
        function createConeAbility(size) {
            const abilityMatrix = Array(size).fill(0).map(() => Array(size).fill(0));
            const center = Math.floor(size / 2);
            for (let r = 0; r < size; r++) {
                for (let c = 0; c < size; c++) {
                    if (r >= Math.abs(c - center)) {
                        abilityMatrix[r][c] = 1;
                    }
                }
            }
            return abilityMatrix;
        }

        function createCrossAbility(size) {
            const abilityMatrix = Array(size).fill(0).map(() => Array(size).fill(0));
            const center = Math.floor(size / 2);
            for (let r = 0; r < size; r++) {
                for (let c = 0; c < size; c++) {
                    if (r === center || c === center) {
                        abilityMatrix[r][c] = 1;
                    }
                }
            }
            return abilityMatrix;
        }

        function createOctahedronAbility(size) {
            const abilityMatrix = Array(size).fill(0).map(() => Array(size).fill(0));
            const center = Math.floor(size / 2);
            for (let r = 0; r < size; r++) {
                for (let c = 0; c < size; c++) {
                    if (Math.abs(r - center) + Math.abs(c - center) <= center) {
                        abilityMatrix[r][c] = 1;
                    }
                }
            }
            return abilityMatrix;
        }

        // Aplica a matriz de habilidade a um tabuleiro específico (usado nas demos)
        function applyAbilityToDemoBoard(abilityMatrix, centerRow, centerCol, targetBoard) {
            const abilitySize = abilityMatrix.length;
            const offset = Math.floor(abilitySize / 2);

            for (let r = 0; r < abilitySize; r++) {
                for (let c = 0; c < abilitySize; c++) {
                    const boardRow = centerRow - offset + r;
                    const boardCol = centerCol - offset + c;

                    if (boardRow >= 0 && boardRow < BOARD_SIZE &&
                        boardCol >= 0 && boardCol < BOARD_SIZE &&
                        abilityMatrix[r][c] === 1 &&
                        targetBoard[boardRow][boardCol] !== SHIP) { // Não sobrepõe navios
                        
                        targetBoard[boardRow][boardCol] = ABILITY_EFFECT;
                    }
                }
            }
        }

        // --- Event Listeners ---
        startGameButton.addEventListener('click', startGame);
        resetGameButton.addEventListener('click', resetGame);
        novatoDemoButton.addEventListener('click', runNovatoDemo);
        aventureiroDemoButton.addEventListener('click', runAventureiroDemo);
        mestreDemoButton.addEventListener('click', runMestreDemo);

        // --- Inicialização do Jogo na Carga da Página ---
        document.addEventListener('DOMContentLoaded', () => {
            resetGame(); // Inicia um novo jogo limpo ao carregar a página
            updateMessage('Bem-vindo à Batalha Naval! Pressione "Iniciar Jogo" para jogar.');
        });
    </script>
</body>
</html>
