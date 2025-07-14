// DevMirror - Backend with JWT Auth + AI + Snippet Manager

package com.devmirror;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DevMirrorApplication {
    public static void main(String[] args) {
        SpringApplication.run(DevMirrorApplication.class, args);
    }
}

// --- Model ---
package com.devmirror.model;

import jakarta.persistence.*;
import java.time.LocalDateTime;

@Entity
public class JournalEntry {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String title;

    @Column(length = 5000)
    private String content;

    private String mood;
    private LocalDateTime timestamp = LocalDateTime.now();

    // Getters and setters
}

@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String username;
    private String password;

    // Getters and setters
}

@Entity
public class CodeSnippet {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String title;
    private String language;
    private String tags;

    @Column(length = 10000)
    private String code;

    private LocalDateTime createdAt = LocalDateTime.now();

    // Getters and setters
}

// --- Repository ---
package com.devmirror.repository;

import com.devmirror.model.JournalEntry;
import com.devmirror.model.User;
import com.devmirror.model.CodeSnippet;
import org.springframework.data.jpa.repository.JpaRepository;

import java.util.Optional;

public interface JournalRepository extends JpaRepository<JournalEntry, Long> {}
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByUsername(String username);
}
public interface SnippetRepository extends JpaRepository<CodeSnippet, Long> {}

// --- Controller ---
package com.devmirror.controller;

import com.devmirror.model.*;
import com.devmirror.repository.*;
import com.devmirror.service.AiService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;
import java.util.List;

@RestController
@RequestMapping("/api/journals")
public class JournalController {
    @Autowired private JournalRepository journalRepository;
    @Autowired private AiService aiService;

    @GetMapping
    public List<JournalEntry> getAll() {
        return journalRepository.findAll();
    }

    @PostMapping
    public JournalEntry create(@RequestBody JournalEntry entry) {
        return journalRepository.save(entry);
    }

    @DeleteMapping("/{id}")
    public void delete(@PathVariable Long id) {
        journalRepository.deleteById(id);
    }

    @PostMapping("/ai/suggest")
    public String getSuggestion(@RequestBody String codeSnippet) {
        return aiService.getSuggestion(codeSnippet);
    }
}

@RestController
@RequestMapping("/api/snippets")
public class SnippetController {
    @Autowired private SnippetRepository snippetRepository;
    @Autowired private AiService aiService;

    @GetMapping
    public List<CodeSnippet> getAll() {
        return snippetRepository.findAll();
    }

    @PostMapping
    public CodeSnippet create(@RequestBody CodeSnippet snippet) {
        return snippetRepository.save(snippet);
    }

    @DeleteMapping("/{id}")
    public void delete(@PathVariable Long id) {
        snippetRepository.deleteById(id);
    }

    @PostMapping("/ai-review")
    public String aiReview(@RequestBody String code) {
        return aiService.getSuggestion(code);
    }
}

@RestController
@RequestMapping("/api/auth")
public class AuthController {
    @Autowired private UserRepository userRepository;
    @Autowired private JwtService jwtService;

    @PostMapping("/register")
    public String register(@RequestBody User user) {
        user.setPassword(jwtService.encodePassword(user.getPassword()));
        userRepository.save(user);
        return "User registered";
    }

    @PostMapping("/login")
    public String login(@RequestBody User user) {
        return userRepository.findByUsername(user.getUsername())
                .filter(u -> jwtService.matches(user.getPassword(), u.getPassword()))
                .map(u -> jwtService.generateToken(user.getUsername()))
                .orElse("Invalid credentials");
    }
}

// --- AI Service ---
package com.devmirror.service;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.http.*;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

@Service
public class AiService {
    @Value("${openai.api.key}")
    private String apiKey;

    public String getSuggestion(String code) {
        RestTemplate restTemplate = new RestTemplate();
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON);
        headers.setBearerAuth(apiKey);

        Map<String, Object> body = new HashMap<>();
        body.put("model", "gpt-3.5-turbo");
        body.put("messages", List.of(
                Map.of("role", "system", "content", "You are a Java code reviewer."),
                Map.of("role", "user", "content", "Please review this code and give suggestions: " + code)
        ));

        HttpEntity<Map<String, Object>> entity = new HttpEntity<>(body, headers);
        ResponseEntity<Map> response = restTemplate.postForEntity("https://api.openai.com/v1/chat/completions", entity, Map.class);

        if (response.getStatusCode() == HttpStatus.OK) {
            Map<String, Object> choice = ((List<Map<String, Object>>) response.getBody().get("choices")).get(0);
            Map<String, Object> message = (Map<String, Object>) choice.get("message");
            return message.get("content").toString();
        }
        return "No suggestion available.";
    }
}

// --- JWT and Security config remain the same ---
