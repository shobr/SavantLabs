import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import okhttp3.*;

import java.io.IOException;
import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

public class GitHubConnector {
    private static final String BASE_URL = "https://api.github.com";
    private static final String TOKEN = System.getenv("GITHUB_TOKEN"); // Export before running

    private static final OkHttpClient client = new OkHttpClient();
    private static final ObjectMapper mapper = new ObjectMapper();

    public static void main(String[] args) throws IOException {
        // Read github username
        Scanner scanner = new Scanner(System.in);
        System.out.print("Enter GitHub username: ");
        String user = scanner.nextLine();

        List<Repository> repositories = fetchRepositories(user);

        for (Repository repo : repositories) {
            List<Commit> commits = fetchCommits(user, repo.getName());
            System.out.println("Repository: " + repo.getName());
            for (Commit commit : commits) {
                System.out.println(commit);
            }
        }
    }

    public static List<Repository> fetchRepositories(String user) throws IOException {
        List<Repository> repositories = new ArrayList<>();
        int page = 1;
        while (true) {
            HttpUrl url = HttpUrl.parse(BASE_URL + "/users/" + user + "/repos")
                    .newBuilder()
                    .addQueryParameter("per_page", "100")
                    .addQueryParameter("page", String.valueOf(page))
                    .build();

            Request request = new Request.Builder()
                    .url(url)
                    .header("Authorization", "token " + TOKEN)
                    .build();

            try (Response response = client.newCall(request).execute()) {
                if (!response.isSuccessful()) {
                    System.err.println("Failed to fetch repositories: " + response.code());
                    break;
                }

                JsonNode node = mapper.readTree(response.body().string());
                if (node.size() == 0) {
                    break;
                }

                for (JsonNode node : node) {
                    repositories.add(new Repository(node.get("name").asText()));
                }
                page++;
            }
        }
        return repositories;
    }

    public static List<Commit> fetchCommits(String user, String repo) throws IOException {
        List<Commit> commits = new ArrayList<>();
        int page = 1;
        while (commits.size() < 20) {
            HttpUrl url = HttpUrl.parse(BASE_URL + "/repos/" + user + "/" + repo + "/commits")
                    .newBuilder()
                    .addQueryParameter("per_page", "20")
                    .addQueryParameter("page", String.valueOf(page))
                    .build();

            Request request = new Request.Builder()
                    .url(url)
                    .header("Authorization", "token " + TOKEN)
                    .build();

            try (Response response = client.newCall(request).execute()) {
                if (!response.isSuccessful()) {
                    System.err.println("Failed to fetch commits for " + repo + ": " + response.code());
                    break;
                }

                JsonNode root = mapper.readTree(response.body().string());
                if (!root.isArray() || root.size() == 0) {
                    break;
                }

                for (JsonNode node : root) {
                    String message = node.get("commit").get("message").asText();
                    String author = node.get("commit").get("author").get("name").asText();
                    String date = node.get("commit").get("author").get("date").asText();
                    commits.add(new Commit(message, author, date));
                    if (commits.size() >= 20) {
                        break;
                    }
                }
                page++;
            }
        }
        return commits;
    }

    public static class Repository {
        private String name;

        public Repository(String name) {
            this.name = name;
        }

        public String getName() {
            return name;
        }
    }

    public static class Commit {
        private String message;
        private String author;
        private String date;

        public Commit(String message, String author, String date) {
            this.message = message;
            this.author = author;
            this.date = date;
        }

        @Override
        public String toString() {
            return "Commit{" +
                    "message='" + message + '\'' +
                    ", author='" + author + '\'' +
                    ", date='" + date + '\'' +
                    '}';
        }
    }
}
