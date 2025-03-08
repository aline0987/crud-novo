<?php
// Conexão com o banco de dados
$host = 'localhost';
$db_name = 'gerenciamento_produtos';
$username = 'root';
$password = '';

try {
    $conn = new PDO("mysql:host=$host;dbname=$db_name", $username, $password);
    $conn->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
} catch (PDOException $e) {
    die("Erro de conexão: " . $e->getMessage());
}

// Função para adicionar ou editar produto
function salvarProduto($conn, $dados) {
    if (isset($dados['id'])) {
        // Editar produto
        $query = "UPDATE produtos SET nome = :nome, descricao = :descricao, preco = :preco, estoque = :estoque WHERE id = :id";
    } else {
        // Adicionar produto
        $query = "INSERT INTO produtos (nome, descricao, preco, estoque) VALUES (:nome, :descricao, :preco, :estoque)";
    }

    $stmt = $conn->prepare($query);
    $stmt->bindParam(':nome', $dados['nome']);
    $stmt->bindParam(':descricao', $dados['descricao']);
    $stmt->bindParam(':preco', $dados['preco']);
    $stmt->bindParam(':estoque', $dados['estoque']);

    if (isset($dados['id'])) {
        $stmt->bindParam(':id', $dados['id']);
    }

    return $stmt->execute();
}

// Função para excluir produto
function excluirProduto($conn, $id) {
    $query = "DELETE FROM produtos WHERE id = :id";
    $stmt = $conn->prepare($query);
    $stmt->bindParam(':id', $id);
    return $stmt->execute();
}

// Função para listar produtos
function listarProdutos($conn) {
    $query = "SELECT * FROM produtos";
    $stmt = $conn->prepare($query);
    $stmt->execute();
    return $stmt->fetchAll(PDO::FETCH_ASSOC);
}

// Função para buscar um produto por ID
function buscarProdutoPorId($conn, $id) {
    $query = "SELECT * FROM produtos WHERE id = :id";
    $stmt = $conn->prepare($query);
    $stmt->bindParam(':id', $id);
    $stmt->execute();
    return $stmt->fetch(PDO::FETCH_ASSOC);
}

// Processar operações CRUD
if ($_SERVER['REQUEST_METHOD'] == 'POST') {
    if (isset($_POST['adicionar']) || isset($_POST['editar'])) {
        $dados = [
            'nome' => $_POST['nome'],
            'descricao' => $_POST['descricao'],
            'preco' => $_POST['preco'],
            'estoque' => $_POST['estoque']
        ];

        if (isset($_POST['id'])) {
            $dados['id'] = $_POST['id'];
        }

        if (salvarProduto($conn, $dados)) {
            header("Location: crud.php");
            exit();
        } else {
            echo "Erro ao salvar produto.";
        }
    }
}

if (isset($_GET['excluir'])) {
    if (excluirProduto($conn, $_GET['excluir'])) {
        header("Location: crud.php");
        exit();
    } else {
        echo "Erro ao excluir produto.";
    }
}

// Verificar se estamos editando um produto
$editar = false;
if (isset($_GET['editar'])) {
    $editar = true;
    $produto = buscarProdutoPorId($conn, $_GET['editar']);
}

// Listar produtos
$produtos = listarProdutos($conn);
?>

<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <title>Gerenciamento de Produtos</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 20px;
        }
        table {
            width: 100%;
            border-collapse: collapse;
        }
        table, th, td {
            border: 1px solid #ddd;
        }
        th, td {
            padding: 8px;
            text-align: left;
        }
        th {
            background-color: #f2f2f2;
        }
        form {
            margin-bottom: 20px;
        }
        label {
            display: block;
            margin-bottom: 5px;
        }
        input, textarea {
            width: 100%;
            padding: 8px;
            margin-bottom: 10px;
            border: 1px solid #ddd;
            border-radius: 4px;
        }
        button {
            padding: 10px 15px;
            background-color: #28a745;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
        button:hover {
            background-color: #218838;
        }
    </style>
</head>
<body>
    <h1>Gerenciamento de Produtos</h1>

    <!-- Formulário para adicionar/editar produto -->
    <form method="post">
        <?php if ($editar): ?>
            <input type="hidden" name="id" value="<?php echo $produto['id']; ?>">
        <?php endif; ?>
        <label for="nome">Nome:</label>
        <input type="text" name="nome" value="<?php echo $editar ? $produto['nome'] : ''; ?>" required>
        <label for="descricao">Descrição:</label>
        <textarea name="descricao" required><?php echo $editar ? $produto['descricao'] : ''; ?></textarea>
        <label for="preco">Preço:</label>
        <input type="number" step="0.01" name="preco" value="<?php echo $editar ? $produto['preco'] : ''; ?>" required>
        <label for="estoque">Estoque:</label>
        <input type="number" name="estoque" value="<?php echo $editar ? $produto['estoque'] : ''; ?>" required>
        <button type="submit" name="<?php echo $editar ? 'editar' : 'adicionar'; ?>">
            <?php echo $editar ? 'Atualizar' : 'Adicionar'; ?>
        </button>
    </form>

    <!-- Tabela de produtos -->
    <table>
        <thead>
            <tr>
                <th>ID</th>
                <th>Nome</th>
                <th>Descrição</th>
                <th>Preço</th>
                <th>Estoque</th>
                <th>Ações</th>
            </tr>
        </thead>
        <tbody>
            <?php foreach ($produtos as $produto): ?>
                <tr>
                    <td><?php echo $produto['id']; ?></td>
                    <td><?php echo $produto['nome']; ?></td>
                    <td><?php echo $produto['descricao']; ?></td>
                    <td><?php echo $produto['preco']; ?></td>
                    <td><?php echo $produto['estoque']; ?></td>
                    <td>
                        <a href="crud.php?editar=<?php echo $produto['id']; ?>">Editar</a>
                        <a href="crud.php?excluir=<?php echo $produto['id']; ?>" onclick="return confirm('Tem certeza?')">Excluir</a>
                    </td>
                </tr>
            <?php endforeach; ?>
        </tbody>
    </table>
</body>
</html>
