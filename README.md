package application; //Demo: inserir dados

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.text.ParseException;
import java.text.SimpleDateFormat;

import db.DB;

public class Program {

	public static void main(String[] args) {

		SimpleDateFormat sdf = new SimpleDateFormat("dd/MM/yyyy");
		
		/*
		 * pra ficar um exemplo mais interesaante eu vou inseriri agora um vendedor, que é o seller, se eu for lá no meu
		 * banco de dados no MySQL Workbench eu vou ver o seller la no tables. então eu indo no meu banco de dados no workbench
		 * botão direito na tabela seller e apertar select rows, vai aparecer a tabela com todos os vendedores já cadastrados.
		 * entãoa gora eu vou fazer uma programação pra inserir um novo vendedor.
		 */
		
		// vou começar criando um objeto do tipo Connection.
		Connection conn = null;
		PreparedStatement st = null;
		
		try {
			conn = DB.getConnection();
			//Primeira coisa eu vou me conectar com o banco.
			st = conn.prepareStatement (
					//esse preparedStatement ele espera como argumento uma String que vai ser o meu comando SQL.
					"INSERT INTO seller "
					+ "(Name, Email, BirthDate, BaseSalary, DepartmentId) "
					//aqui entre parenteses eu tenho que colocar os campos da tabela que eu vou querer preencher.
					+ "VALUES "
					+ "(?, ?, ?, ?, ?)",
					Statement.RETURN_GENERATED_KEYS);
					/*
					 * aqui entre parenteses eu vou colocar os valores que eu vou preencher nesses campos, só que ai esses
					 * valores aqui eu não vou colocar ainda, no caso aqui no JDBC eu vou colcoar uma '?' o '?' vai ser o que
					 * a gente chamamos de PlaceRolder, é um lugar aonde depois você vai colocar o valor, aqui no caso como
					 * eu estou precisando colocar cinco valores(os cinco campos de cima), eu vou colocar cinco '?????'
					 *  
					 *  agora o bacana é, agora eu vou fazer uns comandos pra trocar esse interrogação'?' pelo valor que eu
					 *  quiser. Então pra eu definir que o primeiro '?' vai ser um determinado nome, eu vou chamar o seguinte:
					 * st.setString(), ai você usar o set adequado a sua necessidade, por exemplo aqui o Name ele é uma String
					 * então eu vou usar o setString(1, "Carl Purple"), ai eu vou falar o seguinte eu vou trocar o meu
					 * primeiro interrogação, representado pelo número 1, pelo valor, ai eu vou colocar um valor entre " "
					 * por exemplo o Carl Purple, o nome de uma pessoa, então esse valor Carl Purple, vai entrar no lugar do
					 * primeiro '?', por que eu coloquei ali o 1, de primeiro interrogação.
					 * Agora vamos colocar o Email no lugar do segundo interrogação, usando os mesmos passo s, porem o número 2
					 * representando o 2 interrogação e colocando um Email, e assim por diante, porem com a data de aniversario
					 * eu tenho que instanciar o SimpleDateFormat. Nos estamos acostumados a usar o java.util.Date pra mecher
					 * com data, só que quando você vai instanciar uma data aqui pro jdbc pra jogar no prepareStatement você
					 * tem que colocar o java.sql.Date(). E ai pra data dar certo com o Date do sql, eu tenho que colocar o
					 * .getTime() apos o parenteses com a data, ai eu vou pegar a data: sdf.parse("22/04/1985").getTime() e
					 * esse valor eu vou usar pra instanciar o java.sql.Date().
					 */
			st.setString(1, "Carl Purple");
			st.setString(2, "Carl@gmail.com");
			st.setDate(3, new java.sql.Date(sdf.parse("22/04/1985").getTime()));
			st.setDouble(4, 3000.0);
			st.setInt(5, 4);
			
			/*
			 * como eu faço pra executar esse comando? é só eu chamar st.executeUpdate();, quando é uma operação que eu vou
			 * alterar os dados, você chama o executeUpdate, e ai o resultado dessa operação é um numero inteiro indicando
			 * quantas linhas foram alteradas, então eu criei uma variavel do tipo int chamada rowsAffected e essa variavel 
			 * vai receber o resultado do meu execute update.
			 * essa variavel é só pra eu saber quantas linhas foram alteradas no banco de dados.
			 */
			int rowsAffected = st.executeUpdate();
			
			/*
			 * logo apos criar o comando prepareStatement, no final você coloca Statement.RETURN_GENERATED_KEYS);, esse 
			 * comando faz com que permita que você recupere o ID do novo objeto inserido.
			 * essa função st.getGeneratedKeys(); ela retorna pra gente um objeto do tipo resultSet, então antes dela eu tenho
			 * que declarar um resultSet, recebendo essa chamada de função, e o seguinte esse resultSet pode ter mais de um
			 * valor, no caso desse exemplo eu estou inserido apenas um vendedor, mas eu poderia ter feito uma inserção de 
			 * varios objetos ao mesmo tempo, então isso é possivel tambem, por isso ele retorna o ResultSet com um ou mais
			 * valores.
			 * agora eu vou percorrer esse ResultSet com o while, e pra cada valor eu vou fazer: int id = rs.getInt();
			 * mas nesse caso ao invez de colocar o "Id" ente os parenteses do getInt, eu vou colocar o "1", a posição 1
			 * por que vai ser uma tabelinha auxiliar que vai ter apenas uma coluna contendo os Id's, então eu vou colocar o 
			 * numero 1 pra indicar que eu quero o valor da primeira colouna, então eu vou pegar esse Id na variavel: int id
			 * e ai eu vou mostrar na tela 
			 */
			if (rowsAffected > 0) {
				ResultSet rs = st.getGeneratedKeys();
				while (rs.next()) {
					int id = rs.getInt(1);
					System.out.println("Done! Id = " + id);
				}
			}
			else {
				System.out.println("No rown affected!");
			}
			
		}
		catch (SQLException e) {
			e.printStackTrace();
		}
		catch (ParseException e) {
			e.printStackTrace();
		}
		finally {
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

![1](https://user-images.githubusercontent.com/61166475/155028521-3dbc3ee2-1384-4b93-ac43-9d77ffdc1bdf.png)
