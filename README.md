package application; //Preparação do primeiro projeto no Eclipse

import java.sql.Connection;

import db.DB;

public class Program {

	public static void main(String[] args) {

		Connection conn = DB.getConnection();
		DB.closeConnection();
		
	}

}

===========================================================================

package db;

import java.io.FileInputStream;
import java.io.IOException;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.util.Properties;

public class DB {
	
	private static Connection conn = null;
	/*
	 * um metodo para conectar com os bancos de dados
	 */
	public static Connection getConnection() {
		if (conn == null) { // se ainda estiver nullo, eu vou ter que fazer um codigo para conectar com o banco de dados.
			try {
				/*
				 * pra conectar com o banco a primeira coisa que vou fazer é pegar as propriedades de conexão usando 
				 * Properties props = loadProperties();, pronto, peguei aqui as propriedades do banco de dados
				 * utilizando o loadProperties() la de baixo no final do programa.
				 */
				Properties props = loadProperties();
				String url = props.getProperty("dburl");
				/*
				 * com essas propriedades em maos eu vou criar uma String URL que vai ser o meu url do banco de dados,
				 * como que eu vou pegar essa string? eu vou acessar o meu objeto props e chamar o .getProperty() e entre ()
				 * eu vou chamar o nome da propriedades que é "dburl" porque que é dburl? por que é isso que está definido
				 * no arquivo db.properties.
				 */
				conn = DriverManager.getConnection(url, props);
				/*
				 * e au pra que eu possa obter uma conexão com o meu banco de dados eu vou ter que chamar o
				 * DriverManager.getConnection(url, props); e isso é vou atribuir pra minha variavel de cima chamada conn.
				 * 
				 * então o que fazemos aqui, basicamente a gente pegou as propriedades no caso aqui a url, precisava aqui
				 * pro parametro do metodo tambem e conectamos com o nosso banco de dados utilizando o  
				 * DriverManager.getConnection(url, props);.
				 * conectar com banco de dados aqui no jdbc é instanciar um objeto do tipo Connection, que é o que a gente fez 
				 * usando esse comando aqui: DriverManager.getConnection(url, props);. Perceba que salvei esse objeto nessa
				 * variavel chamada conn, então agora a minha conexão ela está salva, a proxima vez que esse metodo
				 * getConnection for chamado, esse meu test conn == null vai falhar e ele vai simplesmente pular esse if
				 * e retornar a conn  ja existente.
				 */
			}
			catch (SQLException e) {
				throw new DbException(e.getMessage());
			}
		}
		return conn;
	}
	
	// metodo pra fechar a conexão
	public static void closeConnection() {
		if (conn != null) { // se o conn estiver estanciado eu chamo o conn.close();
			try {
				conn.close();
			}
			catch (SQLException e) {
				throw new DbException(e.getMessage());
			}
		}
	}
	/*
	 * metodo auxiliar private porque ele vai ser usado somente aqui nessa classe.
	 * esse aqui então é metodo para carregar as propiedades que estão definidas aqui no arquivo db.properties.
	 */
	private static Properties loadProperties() {
		try(FileInputStream fs = new FileInputStream("db.properties")){
			/*
			 * esse try é pra abrir os arquivos, ai eu instancio esse fs com um new FileInputStream() passando entre parenteses
			 * o parametro que é o nome do arquivo, como meu arquivo está na pasta raiz do projetopasta colocar o nome assim
			 * que vai da certo.
			 */
			Properties props = new Properties();
			/*
			 * muito bem, então a implementação desse try vai ser o seguinte, eu vou instanciar um objeto do tipo Properties
			 * ai eu vou fazer o seguinte, props.load(); recebendo como parametro o fs, esse comando load aqui do objeto
			 * properties, ele faz a leitura do arquivo properties apontado pelo meu FileInputStream fs, e vai guardar
			 * os dados dentro do objeto props.
			 */
			props.load(fs);
			// e no final aqui eu retorno o props
			return props;
			/*
			 * so que aqui essa operação pode dar algum erro, então ta faltando aqui eu tratar uma excessão.
			 */
		}
		catch(IOException e) {
			throw new DbException(e.getMessage());
		}
		
	}
}

===========================================================================

package db;

public class DbException extends RuntimeException {

	private static final long serialVersionUID = 1L;

	public DbException(String msg) {
		super(msg);
	}
}

====================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================

package application; //Demo: recuperar dados

import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

import db.DB;

public class Program {

	public static void main(String[] args) {

		/*
		 * fazer um programa pra gente conectar o banco, preparar uma consular sql que vai ser o Statement pra buscar todos os
		 * departamentos do banco de dados, e o resultado dessa consulta eu vou guardar na minha variavel rs que é o resultSet
		 */
		
		Connection conn = null;
		Statement st = null;
		ResultSet rs = null;
		
		/*
		 * eu vou começar fazendo um bloco try por que todas essas operações elas vao acessar um banco de dados, vai acessar
		 * um recurso externo que pode gerar exceções.
		 */
		try {
			conn = DB.getConnection(); //com isso eu vou conectar o banco de dados.
			
			st = conn.createStatement(); //pra instanciar um statement eu posso chamar a minha var de conn .createStatement.
			
			/*
			 * agora eu vou pegar a minha variavel statement que já está instanciada, vou chamar nela o metodo chamado
			 * executeQuery, esse metodo ele espera que eu entre com uma String, que é justamente o comando SQL
			 * entãoa qui eu vou fazer assim: "select * from department", no caso a minha tabela no Workbech se chama
			 * department, e o resultado dessa minha consulta eu vou atribuir na variavel rs, que é o resultSet.
			 * então na verdade eu vou fazer assim: rs = st.executeQuery("select * from department");
			 */
			rs = st.executeQuery("select * from department");
			/*
			 * isso aqui executa essa consulta lá no banco de dados pra mim, trazendo o resultado pra variavel rs.
			 * 
			 * esse meu rs conforme eu está na imagem1, ele vai ser um objeto que tem aquela forma de tabela, por padrão ele
			 * vai começar na posição ZERO, que é antes de ter os dados, e ai pra percorrer esses dados, eu vou chamar a 
			 * função next(), o next ele vai pra posição 1, 2, 3, 4 e assim por diante.
			 */
			
			while (rs.next()) { //ja que o next ele retorna false caso esteje no ultimo, o while vai funcionar enquanto existir p proximo.
				/*
				 * ai eu vou imprimri oque? o meu rs é um objeto que tem o formato daquela tabela, quando eu fizer o comando
				 * next pela primeira vez, eu vou estar na posição 1 do resultSet, ai como eu faço pra acessar o campo
				 * ID e o campo NAME na posição 1 do meu resultSet? pra acessar um campo do meu resultSet é muito facil
				 * olha só:
				 * pra acessar é só colocar: (rs.getInt("id") + "," rs.getString("name")), colocar entre parenteses assim ó
				 * pega o inteiro que tá lá no campo "Id", e como eu pego o valor do nome? é só eu chamar . rs.getString("Name")
				 * é só colocar o nome do campo no getString.
				 * é facil percorrer o resultSet, é só você fazer um enquanto(while) rs.next(), e ai pra cada posição
				 * que você percorrer você pode dar o getInt, getString, getDouble, conforme a sua necessidade.
				 */
				System.out.println(rs.getInt("Id") + ", " + rs.getString("Name")); 
			}
			
		}
		catch (SQLException e) {
				e.printStackTrace();
		}
		/*
		 * ai como esse recursos Connection, Statement, ResultSet eles são recursos externos que não são controlados
		 * pela JVM do java, é interessante que eu feche esse recurso manualmente pra evitar que o programa tenha algum tipo
		 * de vazamento de memoria, então como eu fecho? eu vou acrescentar aqui depois do catch uma clausula finally, e 
		 * esse finelly eu vou chamar o resultSet.close(); depois o statement.close(); e depois o DB.closeConnection();
		 * que eu ja fiz no jdbc1 lá, pra fechar a conexão....
		 * só que tem um detalhe, tanto o rs quanto o st podem lançar uma excessão SQLException tambem, então é o seguinte, 
		 * pra não precisar ficar botando um try/catch em cada um deles, eu vou na minha classe DB e acrescentar lá metodos
		 * auxiliares pra eu fechar esses objetos no final... ir pra la no final!
		 */
		finally {
			DB.closeResultSet(rs);
			DB.closeStatement(st);
			DB.closeConnection();
		}	
		
	}
}

===========================================================================

package db;

import java.io.FileInputStream;
import java.io.IOException;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.Properties;

public class DB {
	
	private static Connection conn = null;
	/*
	 * um metodo para conectar com os bancos de dados
	 */
	public static Connection getConnection() {
		if (conn == null) { // se ainda estiver nullo, eu vou ter que fazer um codigo para conectar com o banco de dados.
			try {
				/*
				 * pra conectar com o banco a primeira coisa que vou fazer é pegar as propriedades de conexão usando 
				 * Properties props = loadProperties();, pronto, peguei aqui as propriedades do banco de dados
				 * utilizando o loadProperties() la de baixo no final do programa.
				 */
				Properties props = loadProperties();
				String url = props.getProperty("dburl");
				/*
				 * com essas propriedades em maos eu vou criar uma String URL que vai ser o meu url do banco de dados,
				 * como que eu vou pegar essa string? eu vou acessar o meu objeto props e chamar o .getProperty() e entre ()
				 * eu vou chamar o nome da propriedades que é "dburl" porque que é dburl? por que é isso que está definido
				 * no arquivo db.properties.
				 */
				conn = DriverManager.getConnection(url, props);
				/*
				 * e au pra que eu possa obter uma conexão com o meu banco de dados eu vou ter que chamar o
				 * DriverManager.getConnection(url, props); e isso é vou atribuir pra minha variavel de cima chamada conn.
				 * 
				 * então o que fazemos aqui, basicamente a gente pegou as propriedades no caso aqui a url, precisava aqui
				 * pro parametro do metodo tambem e conectamos com o nosso banco de dados utilizando o  
				 * DriverManager.getConnection(url, props);.
				 * conectar com banco de dados aqui no jdbc é instanciar um objeto do tipo Connection, que é o que a gente fez 
				 * usando esse comando aqui: DriverManager.getConnection(url, props);. Perceba que salvei esse objeto nessa
				 * variavel chamada conn, então agora a minha conexão ela está salva, a proxima vez que esse metodo
				 * getConnection for chamado, esse meu test conn == null vai falhar e ele vai simplesmente pular esse if
				 * e retornar a conn  ja existente.
				 */
			}
			catch (SQLException e) {
				throw new DbException(e.getMessage());
			}
		}
		return conn;
	}
	
	// metodo pra fechar a conexão
	public static void closeConnection() {
		if (conn != null) { // se o conn estiver estanciado eu chamo o conn.close();
			try {
				conn.close();
			}
			catch (SQLException e) {
				throw new DbException(e.getMessage());
			}
		}
	}
	/*
	 * metodo auxiliar private porque ele vai ser usado somente aqui nessa classe.
	 * esse aqui então é metodo para carregar as propiedades que estão definidas aqui no arquivo db.properties.
	 */
	private static Properties loadProperties() {
		try(FileInputStream fs = new FileInputStream("db.properties")){
			/*
			 * esse try é pra abrir os arquivos, ai eu instancio esse fs com um new FileInputStream() passando entre parenteses
			 * o parametro que é o nome do arquivo, como meu arquivo está na pasta raiz do projetopasta colocar o nome assim
			 * que vai da certo.
			 */
			Properties props = new Properties();
			/*
			 * muito bem, então a implementação desse try vai ser o seguinte, eu vou instanciar um objeto do tipo Properties
			 * ai eu vou fazer o seguinte, props.load(); recebendo como parametro o fs, esse comando load aqui do objeto
			 * properties, ele faz a leitura do arquivo properties apontado pelo meu FileInputStream fs, e vai guardar
			 * os dados dentro do objeto props.
			 */
			props.load(fs);
			// e no final aqui eu retorno o props
			return props;
			/*
			 * so que aqui essa operação pode dar algum erro, então ta faltando aqui eu tratar uma excessão.
			 */
		}
		catch(IOException e) {
			throw new DbException(e.getMessage());
		}
	}
	//vou criar um metodo pra fechar um objeto do tipo statement.
	public static void closeStatement(Statement st) {
		/*
		 * então eu vou implementar um metodo pra fechar um statement st, como vou fechar esse metodo? basicamente eu vou 
		 * fazer o seguinte, se esse st foi diferente de null, ou seja, se ele estiver instanciado, eu vou chamar o st.close()
		 * só que esse st.close() pode gerar excessão SQLExeption, então o que eu vou fazer? eu vou aceitar a correção
		 * Surround with try/catch, so que agora no caso de acontecer uma excessão, eu vou chamar o meu DbException 
		 * passando a mensagem da excessão SQLException.
		 * por que? por que ai eu vou poder ter aqui o fechamento de statement só que lançando uma excessão RuntimeException
		 * então não vai ser obrigado a ficar tratando isso toda hora com try/catch, então no programa principal, no lugar de
		 * colocar o st.close, eu vou colocar o DB.closeStatement(st);
		 * ai depois é só fazer a mesma coisa com o resultSet.
		 */
		if (st != null) {
			try {
				st.close();
			}
			catch (SQLException e) {
				throw new DbException(e.getMessage());
			}
		}
	}
	
	public static void closeResultSet(ResultSet rs) {
		if (rs != null) {
			try {
				rs.close();
			}
			catch (SQLException e) {
				throw new DbException(e.getMessage());
			}
		}
	}
}

===========================================================================

package db;

public class DbException extends RuntimeException {

	private static final long serialVersionUID = 1L;

	public DbException(String msg) {
		super(msg);
	}
}

![1](https://user-images.githubusercontent.com/61166475/155030111-a1abc282-571d-4656-bf52-a5baeb53fc69.png)
