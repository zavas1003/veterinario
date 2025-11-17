# veterinario
import java.sql.*;

public class SistemaVeterinario {
    
    // Função para conectar ao banco de dados SQLite
    public static Connection connect() throws SQLException {
        // Conectar ao banco de dados SQLite (cria o banco se não existir)
        String url = "jdbc:sqlite:sistema_veterinario.db";
        return DriverManager.getConnection(url);
    }
    
    // Função para criar as tabelas
    public static void createTables(Connection conn) throws SQLException {
        Statement stmt = conn.createStatement();
        
        // Criar a tabela tblspecies
        String createTblSpecies = "CREATE TABLE IF NOT EXISTS tblspecies ("
                + "espid INTEGER PRIMARY KEY, "
                + "espnome TEXT)";
        stmt.execute(createTblSpecies);
        
        // Criar a tabela tblanimals
        String createTblAnimals = "CREATE TABLE IF NOT EXISTS tblanimals ("
                + "aniid INTEGER PRIMARY KEY, "
                + "animnome TEXT, "
                + "anipapelido TEXT, "
                + "anidatnasc DATE, "
                + "anobsservacoes TEXT, "
                + "espid INTEGER, "
                + "FOREIGN KEY (espid) REFERENCES tblspecies(espid))";
        stmt.execute(createTblAnimals);
        
        // Criar a tabela tblclients
        String createTblClients = "CREATE TABLE IF NOT EXISTS tblclients ("
                + "cliid INTEGER PRIMARY KEY, "
                + "clinome TEXT, "
                + "clcpf TEXT, "
                + "clemail TEXT, "
                + "clidatacadastro DATE)";
        stmt.execute(createTblClients);
        
        // Criar a tabela tblanimalsclients
        String createTblAnimalsClients = "CREATE TABLE IF NOT EXISTS tblanimalsclients ("
                + "cliid INTEGER, "
                + "aniid INTEGER, "
                + "FOREIGN KEY (cliid) REFERENCES tblclients(cliid), "
                + "FOREIGN KEY (aniid) REFERENCES tblanimals(aniid), "
                + "PRIMARY KEY (cliid, aniid))";
        stmt.execute(createTblAnimalsClients);
        
        stmt.close();
    }
    
    // Função para inserir dados nas tabelas
    public static void insertData(Connection conn) throws SQLException {
        // Inserir dados na tabela tblspecies
        String insertSpecies = "INSERT INTO tblspecies (espid, espnome) VALUES (?, ?)";
        try (PreparedStatement pstmt = conn.prepareStatement(insertSpecies)) {
            pstmt.setInt(1, 1);
            pstmt.setString(2, "Cachorro");
            pstmt.executeUpdate();
            
            pstmt.setInt(1, 2);
            pstmt.setString(2, "Gato");
            pstmt.executeUpdate();
        }
        
        // Inserir dados na tabela tblanimals
        String insertAnimals = "INSERT INTO tblanimals (aniid, animnome, anipapelido, anidatnasc, anobsservacoes, espid) VALUES (?, ?, ?, ?, ?, ?)";
        try (PreparedStatement pstmt = conn.prepareStatement(insertAnimals)) {
            pstmt.setInt(1, 1);
            pstmt.setString(2, "Rex");
            pstmt.setString(3, "Rexinho");
            pstmt.setString(4, "2022-05-01");
            pstmt.setString(5, "Cachorro muito ativo.");
            pstmt.setInt(6, 1);
            pstmt.executeUpdate();
            
            pstmt.setInt(1, 2);
            pstmt.setString(2, "Miau");
            pstmt.setString(3, "Miadinho");
            pstmt.setString(4, "2021-03-14");
            pstmt.setString(5, "Gato calmo e tranquilo.");
            pstmt.setInt(6, 2);
            pstmt.executeUpdate();
        }
        
        // Inserir dados na tabela tblclients
        String insertClients = "INSERT INTO tblclients (cliid, clinome, clcpf, clemail, clidatacadastro) VALUES (?, ?, ?, ?, ?)";
        try (PreparedStatement pstmt = conn.prepareStatement(insertClients)) {
            pstmt.setInt(1, 1);
            pstmt.setString(2, "Carlos Silva");
            pstmt.setString(3, "12345678901");
            pstmt.setString(4, "carlos@email.com");
            pstmt.setString(5, "2025-11-17");
            pstmt.executeUpdate();
        }
        
        // Relacionar clientes aos animais na tabela tblanimalsclients
        String insertAnimalsClients = "INSERT INTO tblanimalsclients (cliid, aniid) VALUES (?, ?)";
        try (PreparedStatement pstmt = conn.prepareStatement(insertAnimalsClients)) {
            pstmt.setInt(1, 1);  // Carlos Silva
            pstmt.setInt(2, 1);  // Rex
            pstmt.executeUpdate();
            
            pstmt.setInt(1, 1);  // Carlos Silva
            pstmt.setInt(2, 2);  // Miau
            pstmt.executeUpdate();
        }
    }
    
    // Função para consultar os animais de um cliente
    public static void queryAnimalsOfClient(Connection conn) throws SQLException {
        String query = "SELECT tblanimals.animnome, tblspecies.espnome "
                + "FROM tblanimals "
                + "JOIN tblspecies ON tblanimals.espid = tblspecies.espid "
                + "JOIN tblanimalsclients ON tblanimals.aniid = tblanimalsclients.aniid "
                + "JOIN tblclients ON tblanimalsclients.cliid = tblclients.cliid "
                + "WHERE tblclients.cliid = 1";
        
        try (Statement stmt = conn.createStatement(); ResultSet rs = stmt.executeQuery(query)) {
            while (rs.next()) {
                String animalName = rs.getString("animnome");
                String speciesName = rs.getString("espnome");
                System.out.println("Nome do Animal: " + animalName + ", Espécie: " + speciesName);
            }
        }
    }
    
    public static void main(String[] args) {
        try (Connection conn = connect()) {
            // Criar as tabelas
            createTables(conn);
            
            // Inserir dados
            insertData(conn);
            
            // Consultar e exibir os animais de um cliente
            queryAnimalsOfClient(conn);
            
        } catch (SQLException e) {
            System.out.println(e.getMessage());
        }
    }
}
