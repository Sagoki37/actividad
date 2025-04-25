import java.util.*;
import java.util.concurrent.*;
import java.util.regex.*;
import java.nio.file.*;
import java.io.IOException;

public class ValidadorConcurrenteConRegistro {

    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        ExecutorService executor = Executors.newFixedThreadPool(3);
        List<Future<String>> resultados = new ArrayList<>();

        System.out.print("¿Cuántas contraseñas desea validar? ");
        int n = sc.nextInt();
        sc.nextLine(); // limpiar buffer

        for (int i = 1; i <= n; i++) {
            System.out.print("Ingrese la contraseña #" + i + ": ");
            String password = sc.nextLine();
            String nombre = "Contraseña #" + i;

            // Lambda con Callable para retornar resultado
            Callable<String> tarea = () -> {
                boolean esValida = validar(password);
                String resultado = nombre + " => " + password + " : " + (esValida ? "✅ VÁLIDA" : "❌ INVÁLIDA");
                System.out.println("🔍 " + resultado);
                return resultado;
            };

            resultados.add(executor.submit(tarea));
        }

        executor.shutdown();

        // Esperar y escribir resultados en archivo
        List<String> lineasParaArchivo = new ArrayList<>();
        for (Future<String> future : resultados) {
            try {
                lineasParaArchivo.add(future.get());
            } catch (InterruptedException | ExecutionException e) {
                e.printStackTrace();
            }
        }

        Path archivo = Paths.get("log.txt");
        try {
            Files.write(archivo, lineasParaArchivo, StandardOpenOption.CREATE, StandardOpenOption.APPEND);
            System.out.println("📝 Resultados registrados en: log.txt");
        } catch (IOException e) {
            System.err.println("❌ Error al escribir el archivo de registro: " + e.getMessage());
        }
    }

    // 🔐 Método que valida la contraseña
    public static boolean validar(String pwd) {
        if (pwd.length() < 8) return false;

        Pattern mayus = Pattern.compile("[A-Z]");
        Pattern minus = Pattern.compile("[a-z]");
        Pattern num = Pattern.compile("\\d");
        Pattern esp = Pattern.compile("[!@#$%^&*(),.?\":{}|<>]");

        return contar(mayus, pwd) >= 2 &&
               contar(minus, pwd) >= 3 &&
               contar(num, pwd) >= 1 &&
               contar(esp, pwd) >= 1;
    }

    public static int contar(Pattern patron, String texto) {
        Matcher matcher = patron.matcher(texto);
        return (int) matcher.results().count();  // ✅ expresión lambda moderna
    }
}
