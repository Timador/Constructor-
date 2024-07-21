# Constructor-

    package demo_jdbc.models;
    import javafx.beans.property.SimpleIntegerProperty;
    import javafx.beans.property.SimpleStringProperty;
  
    public class Constructor {
        private final SimpleIntegerProperty constructorId;
        private final SimpleStringProperty name;
    
        public Constructor(int constructorId, String name) {
            this.constructorId = new SimpleIntegerProperty(constructorId);
            this.name = new SimpleStringProperty(name);
        }
    
        public int getConstructorId() {
            return constructorId.get();
        }
  
        public SimpleIntegerProperty constructorIdProperty() {
            return constructorId;
        }
    
        public String getName() {
            return name.get();
        }
    
        public SimpleStringProperty nameProperty() {
            return name;
        }
    }

# ConstructorResult-

    package demo_jdbc.respositories;
    
    public class ConstructorResult {
    
        private String constructorName;
        private int wins;
        private int totalPoints;
        private int rank;
    
        public ConstructorResult(String constructorName, int wins, int totalPoints, int rank) {
            this.constructorName = constructorName;
            this.wins = wins;
            this.totalPoints = totalPoints;
            this.rank = rank;
        }

        public String getConstructorName() {
            return constructorName;
        }
    
        public void setConstructorName(String constructorName) {
            this.constructorName = constructorName;
        }
    
        public int getWins() {
            return wins;
        }
    
        public void setWins(int wins) {
            this.wins = wins;
        }
    
        public int getTotalPoints() {
            return totalPoints;
        }
    
        public void setTotalPoints(int totalPoints) {
            this.totalPoints = totalPoints;
        }
    
        public int getRank() {
            return rank;
        }
    
        public void setRank(int rank) {
            this.rank = rank;
        }
    }


# MAIN-

    package demo_jdbc;
    
    import java.sql.Connection;
    import java.sql.DriverManager;
    import java.sql.ResultSet;
    import java.sql.Statement;
    
    import demo_jdbc.respositories.ConstructorResult;
    import demo_jdbc.respositories.DriverResult;
    import javafx.application.Application;
    import javafx.collections.FXCollections;
    import javafx.collections.ObservableList;
    import javafx.geometry.Pos;
    import javafx.scene.Scene;
    import javafx.scene.control.ComboBox;
    import javafx.scene.control.TableColumn;
    import javafx.scene.control.TableView;
    import javafx.scene.control.cell.PropertyValueFactory;
    import javafx.scene.layout.BorderPane;
    import javafx.scene.layout.VBox;
    import javafx.stage.Stage;
    
    public class Main extends Application {
        private final TableView<DriverResult> driverTableView = new TableView<>();
        private final TableView<ConstructorResult> constructorTableView = new TableView<>();
        private final ObservableList<DriverResult> driverResults = FXCollections.observableArrayList();
        private final ObservableList<ConstructorResult> constructorResults = FXCollections.observableArrayList();
        private final ComboBox<String> typeComboBox = new ComboBox<>();
    
        @Override
        public void start(Stage primaryStage) throws Exception {
            BorderPane borderPane = new BorderPane();
    
            // Configurar ComboBox para elegir tipo de resultados
            typeComboBox.getItems().addAll("Pilotos", "Constructores");
            typeComboBox.setValue("Pilotos"); // Valor por defecto
            typeComboBox.setOnAction(event -> {
                String selectedType = typeComboBox.getValue();
                if (selectedType.equals("Pilotos")) {
                    loadDriverResults(); // Cargar resultados generales de pilotos
                    borderPane.setCenter(driverTableView);
                } else if (selectedType.equals("Constructores")) {
                    loadConstructorResults(); // Cargar resultados generales de constructores
                    borderPane.setCenter(constructorTableView);
                }
            });
            typeComboBox.setStyle("-fx-alignment: CENTER;");
    
            // Crear columnas de TableView para pilotos
            TableColumn<DriverResult, String> driverNameColumn = new TableColumn<>("Nombre del Piloto");
            driverNameColumn.setCellValueFactory(new PropertyValueFactory<>("driverName"));
    
            TableColumn<DriverResult, Integer> driverWinsColumn = new TableColumn<>("Victorias");
            driverWinsColumn.setCellValueFactory(new PropertyValueFactory<>("wins"));
    
            TableColumn<DriverResult, Integer> driverPointsColumn = new TableColumn<>("Puntos Totales");
            driverPointsColumn.setCellValueFactory(new PropertyValueFactory<>("totalPoints"));
    
            TableColumn<DriverResult, Integer> driverRankColumn = new TableColumn<>("Ranking");
            driverRankColumn.setCellValueFactory(new PropertyValueFactory<>("rank"));
    
            driverTableView.getColumns().addAll(driverNameColumn, driverWinsColumn, driverPointsColumn, driverRankColumn);
            driverTableView.setItems(driverResults);
            driverTableView.setColumnResizePolicy(TableView.CONSTRAINED_RESIZE_POLICY);
    
            // Crear columnas de TableView para constructores
            TableColumn<ConstructorResult, String> constructorNameColumn = new TableColumn<>("Nombre del Constructor");
            constructorNameColumn.setCellValueFactory(new PropertyValueFactory<>("constructorName"));
    
            TableColumn<ConstructorResult, Integer> constructorWinsColumn = new TableColumn<>("Victorias");
            constructorWinsColumn.setCellValueFactory(new PropertyValueFactory<>("wins"));
    
            TableColumn<ConstructorResult, Integer> constructorPointsColumn = new TableColumn<>("Puntos Totales");
            constructorPointsColumn.setCellValueFactory(new PropertyValueFactory<>("totalPoints"));
    
            TableColumn<ConstructorResult, Integer> constructorRankColumn = new TableColumn<>("Ranking");
            constructorRankColumn.setCellValueFactory(new PropertyValueFactory<>("rank"));
    
            constructorTableView.getColumns().addAll(constructorNameColumn, constructorWinsColumn, constructorPointsColumn, constructorRankColumn);
            constructorTableView.setItems(constructorResults);
            constructorTableView.setColumnResizePolicy(TableView.CONSTRAINED_RESIZE_POLICY);
    
            // Crear layout y agregar componentes
            VBox vbox = new VBox();
            vbox.setAlignment(Pos.CENTER);
            vbox.getChildren().addAll(typeComboBox);
    
            borderPane.setTop(vbox);
            borderPane.setCenter(driverTableView); // Mostrar inicialmente resultados de pilotos
    
            // Cargar resultados generales por defecto
            loadDriverResults(); // Cargar resultados generales de pilotos
            loadConstructorResults(); // Cargar resultados generales de constructores
    
            // Crear la escena y mostrarla
            Scene scene = new Scene(borderPane, 800, 600);
            primaryStage.setTitle("Resultados de Pilotos y Constructores");
            primaryStage.setScene(scene);
            primaryStage.show();
        }
    
        private void loadDriverResults() {
            driverResults.clear();
            String url = "jdbc:postgresql://localhost:2003/formula1";
            String user = "postgres";
            String password = "postgresql.2003";
            String query = "SELECT * FROM ("
                    + " SELECT d.forename || ' ' || d.surname AS driver_name, "
                    + " COUNT(CASE WHEN res.position = 1 THEN 1 END) AS wins, "
                    + " SUM(res.points) AS total_points, "
                    + " ROW_NUMBER() OVER (ORDER BY SUM(res.points) DESC, COUNT(CASE WHEN res.position = 1 THEN 1 END) DESC) AS rank "
                    + " FROM results res "
                    + " JOIN drivers d ON res.driverid = d.driverid "
                    + " GROUP BY d.driverid, d.forename, d.surname"
                    + ") AS ranked_results "
                    + "ORDER BY total_points DESC;";
    
            try {
                Class.forName("org.postgresql.Driver");
                Connection conn = DriverManager.getConnection(url, user, password);
                Statement stmt = conn.createStatement();
                ResultSet rs = stmt.executeQuery(query);
    
                while (rs.next()) {
                    String driverName = rs.getString("driver_name");
                    int wins = rs.getInt("wins");
                    int totalPoints = rs.getInt("total_points");
                    int rank = rs.getInt("rank");
    
                    DriverResult result = new DriverResult(driverName, wins, totalPoints, rank);
                    driverResults.add(result);
                }
    
                rs.close();
                stmt.close();
                conn.close();
            } catch (Exception e) {
                e.printStackTrace();
            }
    
            // Actualizar TableView
            driverTableView.setItems(driverResults);
        }
    
        private void loadConstructorResults() {
            constructorResults.clear();
            String url = "jdbc:postgresql://localhost:2003/formula1";
            String user = "postgres";
            String password = "postgresql.2003";
            String query = "SELECT * FROM ("
                    + " SELECT c.name AS constructor_name, "
                    + " COUNT(CASE WHEN res.position = 1 THEN 1 END) AS wins, "
                    + " SUM(res.points) AS total_points, "
                    + " ROW_NUMBER() OVER (ORDER BY SUM(res.points) DESC, COUNT(CASE WHEN res.position = 1 THEN 1 END) DESC) AS rank "
                    + " FROM results res "
                    + " JOIN constructor c ON res.constructorid = c.constructorid "
                    + " GROUP BY c.constructorid, c.name"
                    + ") AS ranked_results "
                    + "ORDER BY total_points DESC;";
    
            try {
                Class.forName("org.postgresql.Driver");
                Connection conn = DriverManager.getConnection(url, user, password);
                Statement stmt = conn.createStatement();
                ResultSet rs = stmt.executeQuery(query);
    
                while (rs.next()) {
                    String constructorName = rs.getString("constructor_name");
                    int wins = rs.getInt("wins");
                    int totalPoints = rs.getInt("total_points");
                    int rank = rs.getInt("rank");
    
                    ConstructorResult result = new ConstructorResult(constructorName, wins, totalPoints, rank);
                    constructorResults.add(result);
                }
    
                rs.close();
                stmt.close();
                conn.close();
            } catch (Exception e) {
                e.printStackTrace();
            }
    
            // Actualizar TableView
            constructorTableView.setItems(constructorResults);
        }
    
        public static void main(String[] args) {
            launch(args);
        }
    }



![Imagen de WhatsApp 2024-07-21 a las 15 32 36_38a75021](https://github.com/user-attachments/assets/3399ee2f-b20a-4337-b9bd-618ff37e48f6)

