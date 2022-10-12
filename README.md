# login-e-cadastro-PHP-e-MySQL



Criando um sistema de cadastro e login com PHP e MySQL
Neste artigo vamos aprender a criar um sistema de cadastro e login bem simples, utilizando PHP e MySQL. Vamos ver também alguns conceitos de criptografia md5 e cookies.
69


Anotar
 
Marcar como concluído
Artigos
PHP
Criando um sistema de cadastro e login com PHP e MySQL
Olá, hoje vamos aprender como criar um simples sistema de cadastro e login utilizando PHP e MySQL.

Saiba mais: MySQL Workbench

Lembrando que você deverá ter instalado na sua máquina o MySQL e ter um servidor local com suporte a PHP para poder testar o sistema. (Eu utilizei o MySQL Workbench 5.2 CE + IIS (Com o PHP 5)).
Criando a tabela 'usuarios'
A primeira coisa a ser feita é criar a sua tabela no banco de dados, aqui nós utilizaremos apenas 3 campos (você poderá utilizar quantos campos quiser, desde que , respeite alguma regras que serão explicadas).

Vamos ao código:

Create table usuarios (
ID Int UNSIGNED ZEROFILL NOT NULL AUTO_INCREMENT,
login Varchar(30),
senha Varchar(40),
Primary Key (ID)) ENGINE = MyISAM;
Listagem 1. Tabela usuarios (usuarios.sql)
O código acima, criou uma tabela chamada usuarios, contendo os seguintes campos: login (do tipo varchar com no máximo 30 caracteres), senha (do tipo varchar com no máximo 40 caracteres), e o campo ID (do tipo INT, e com auto incremento.)

Agora segue uma pequena explicação sobre cada 1 dos 3 campos.

Campo 'login' - Armazenará o nome digitado no campo 'login' do formulário (Esse campo não pode se repetir e não podem ser nulos, porém pode ser alterado pelo usuário caso você deseje permitir.) a restrição será aplicada via PHP.
Campo 'senha' - Armazenará o valor da variável senha(que veremos mais a frente)
Campo 'ID' - Armazena um código único de usuário, gerado cada vez que alguém se cadastra, dando identidade única a cada login (Esse código não se repete, e não pode ser alterado, podendo causar falhas como invalidez das contas devido ao compartilhamento de ids)
Criando o Formulário de cadastro
Agora vamos criar o nosso formulário html. Nesse caso ão utilizaremos estilos CSS, apenas um layout simples para entender o funcionamento do sistema (Caso queira saber como estilizar o seu formulário acesse este link: Customizando formulários com CSS )

Vamos ao código apresentado na Listagem 2.

<html>
<head>
<title> Cadastro de Usuário </title>
</head>
<body>
<form method="POST" action="cadastro.php">
<label>Login:</label><input type="text" name="login" id="login"><br>
<label>Senha:</label><input type="password" name="senha" id="senha"><br>
<input type="submit" value="Cadastrar" id="cadastrar" name="cadastrar">
</form>
</body>
</html>
Listagem 2. Página cadastro.html
Cadastro HTML
Figura 1. cadastro.html
Atenção para um ponto importante dessa página:

Estamos utilizando o método post, para o envio dos dados para a página php. Você também pode utilizar o GET, porém, o método POST é mais seguro no envio das informações, já que o método GET passa os valores dos campos como parâmetros pela URL.

Criando o cadastro.php
Agora vamos criar a página cadastro.php, essa página ira tratar os dados inseridos no formulário de cadastro e inserí-los na sua tabela no banco de dados.

Vamos ao código da Listagem 3.

<?php

$login = $_POST['login'];
$senha = MD5($_POST['senha']);
$connect = mysql_connect('nome_do_servidor','nome_de_usuario','senha');
$db = mysql_select_db('nome_do_banco_de_dados');
$query_select = "SELECT login FROM usuarios WHERE login = '$login'";
$select = mysql_query($query_select,$connect);
$array = mysql_fetch_array($select);
$logarray = $array['login'];

  if($login == "" || $login == null){
    echo"<script language='javascript' type='text/javascript'>
    alert('O campo login deve ser preenchido');window.location.href='
    cadastro.html';</script>";

    }else{
      if($logarray == $login){

        echo"<script language='javascript' type='text/javascript'>
        alert('Esse login já existe');window.location.href='
        cadastro.html';</script>";
        die();

      }else{
        $query = "INSERT INTO usuarios (login,senha) VALUES ('$login','$senha')";
        $insert = mysql_query($query,$connect);

        if($insert){
          echo"<script language='javascript' type='text/javascript'>
          alert('Usuário cadastrado com sucesso!');window.location.
          href='login.html'</script>";
        }else{
          echo"<script language='javascript' type='text/javascript'>
          alert('Não foi possível cadastrar esse usuário');window.location
          .href='cadastro.html'</script>";
        }
      }
    }
?>
Listagem 3. Página cadastro.php
Nota: Na linha 7 do código acima, temos a verificação ($login == "" || $login == null), essa é a restrição que falamos anteriormente, em que , o campo login não pode estar nulo.
A verificação na linha 13 refere-se à existência de um usuário com o mesmo nome, ou seja, caso o nome exista no banco de dados, um alerta é acionado.

Criando o formulário de login
Com a página de cadastro criada, vamos agora ao formulário de login, que fará o envio das informações para uma página PHP que verificará a existência desse usuário.

Vamos ao código da Listagem 4.

<html>
<head>
<title> Login de Usuário </title>
</head>
<body>
<form method="POST" action="login.php">
<label>Login:</label><input type="text" name="login" id="login"><br>
<label>Senha:</label><input type="password" name="senha" id="senha"><br>
<input type="submit" value="entrar" id="entrar" name="entrar"><br>
<a href="cadastro.html">Cadastre-se</a>
</form>
</body>
</html>
Listagem 4. - login.html
Login HTML
Figura 2. login.html
Criando o login.php
Essa página será responsável por tratar as informações inseridas na página de login e verificar se o usuário inseriu as informações corretas, e logo em seguida redirecioná-lo.

<?php
$login = $_POST['login'];
$entrar = $_POST['entrar'];
$senha = md5($_POST['senha']);
$connect = mysql_connect('nome_do_servidor','nome_de_usuario','senha');
$db = mysql_select_db('nome_do_banco_de_dados');
  if (isset($entrar)) {

    $verifica = mysql_query("SELECT * FROM usuarios WHERE login =
    '$login' AND senha = '$senha'") or die("erro ao selecionar");
      if (mysql_num_rows($verifica)<=0){
        echo"<script language='javascript' type='text/javascript'>
        alert('Login e/ou senha incorretos');window.location
        .href='login.html';</script>";
        die();
      }else{
        setcookie("login",$login);
        header("Location:index.php");
      }
  }
?>
Listagem 5. login.php
Criando a index.php
Agora criaremos a página index.php , que terá duas partes , uma parte pública, e outra restrita.

<?php
  $login_cookie = $_COOKIE['login'];
    if(isset($login_cookie)){
      echo"Bem-Vindo, $login_cookie <br>";
      echo"Essas informações <font color='red'>PODEM</font> ser acessadas por você";
    }else{
      echo"Bem-Vindo, convidado <br>";
      echo"Essas informações <font color='red'>NÃO PODEM</font> ser acessadas por você";
      echo"<br><a href='login.html'>Faça Login</a> Para ler o conteúdo";
    }
?>
Listagem 6. index.php
Pronto, agora você já tem um simples sistema de cadastro e login para usar no seu projeto, lembrando que isso é apenas o básico e podem ser utilizadas várias outras técnicas para validação de login e segurança.

Links Úteis
Primeiros passos com a jQuery: Aprenda neste curso front-end como criar botões com a biblioteca jQuery.
.NET na prática: Neste Guia de Consulta você encontrará conteúdos que abordam na prática o desenvolvimento de aplicações em .NET utilizando seus variados frameworks, como ASP.NET MVC, Web API e Entity Framework.
Mapeamento Objeto-Relacional com TMS Aurelius: Veremos neste artigo como funciona e como utilizar o framework de Mapeamento Objeto-Relacional TMS Aurelius, que permite lidar com bancos de dados aproveitando os recursos da orientação a objetos.
Saiba mais sobre PHP e MySQL ;)
Guias PHP: Encontre aqui todos os Guias de estudo que você precisa para dominar a linguagem PHP, desde o recurso mais básico até o mais avançado. Escolha o seu!
MySQL: Neste guia de consulta você encontrará diversos conteúdos que podem ser usados ao longo dos seus estudos sobre o banco de dados MySQL. Consulte este guia para aprender mais sobre a administração e uso desse SGBD.
Banco de Dados para Programadores: Neste guia você encontrará os principais conteúdos que você precisa estudar, como desenvolvedor, para trabalhar com bancos de dados.
Tecnologias:
MySQL PHP SQL


