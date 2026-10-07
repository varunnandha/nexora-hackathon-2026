#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <stdbool.h>

#define MAX_BUF 16384

// --- 1. CODEBASE READ & UNDERSTAND ---
char *read_file(const char *filepath) {
    FILE *f = fopen(filepath, "r");
    if (!f) return NULL;

    char *buffer = (char *)malloc(MAX_BUF);
    if (!buffer) { fclose(f); return NULL; }

    size_t len = fread(buffer, 1, MAX_BUF - 1, f);
    buffer[len] = '\0';
    fclose(f);
    return buffer;
}

// --- 2. CODE MODIFICATION / BUG FIXING ---
bool apply_edit(const char *filepath, const char *new_code) {
    FILE *f = fopen(filepath, "w");
    if (!f) return false;

    fputs(new_code, f);
    fclose(f);
    return true;
}

// Backup file before applying agent edits
bool create_backup(const char *filepath, const char *backup_path) {
    char *content = read_file(filepath);
    if (!content) return false;

    bool result = apply_edit(backup_path, content);
    free(content);
    return result;
}

// Restore original code if regression/test failure is detected
void rollback_changes(const char *filepath, const char *backup_path) {
    char *original_code = read_file(backup_path);
    if (original_code) {
        apply_edit(filepath, original_code);
        free(original_code);
    }
    remove(backup_path);
}

// --- 3. TEST EXECUTION & VERIFICATION ---
int execute_test_suite(const char *test_cmd) {
    printf("\n[Agent Test Runner] Executing: %s\n", test_cmd);
    return system(test_cmd); // Returns 0 on success (Pass), non-zero on failure
}

// --- MAIN AI SE AGENT PIPELINE ---
int main() {
    const char *target_file = "app.c";
    const char *backup_file = "app.c.bak";
    
    // Command to compile and run the codebase test suite (Windows compatible)
    const char *test_cmd = "gcc -o app.exe app.c && app.exe";

    // 1. Initial baseline codebase file
    const char *initial_codebase = 
        "#include <stdio.h>\n"
        "int main() {\n"
        "    printf(\"Running existing test suite... Baseline OK!\\n\");\n"
        "    return 0;\n"
        "}\n";

    // 2. Requested Fix / Modification (e.g., Fix authentication boundary condition)
    const char *agent_modified_code = 
        "#include <stdio.h>\n"
        "int authenticate_user(int user_id) {\n"
        "    // Fixed bug: check boundary (0 <= user_id <= 100)\n"
        "    if (user_id >= 0 && user_id <= 100) return 1;\n"
        "    return 0;\n"
        "}\n"
        "int main() {\n"
        "    if (authenticate_user(50) == 1) {\n"
        "        printf(\"Auth boundary test passed!\\n\");\n"
        "        return 0;\n"
        "    }\n"
        "    return 1;\n"
        "}\n";

    printf("=========================================\n");
    printf("   AI Software Engineering Agent Active  \n");
    printf("=========================================\n");

    // Initialize sample repository file
    apply_edit(target_file, initial_codebase);

    // STEP A: Verify baseline tests pass before making any edits
    printf("\n[Step 1] Checking existing codebase integrity...\n");
    if (execute_test_suite(test_cmd) != 0) {
        printf("ERROR: Base codebase tests failed. Agent aborted to avoid breaking working code.\n");
        return 1;
    }
    printf("Integrity Check Passed: Base tests are working.\n");

    // STEP B: Backup target file and apply AI agent modifications
    printf("\n[Step 2] Backing up '%s' and applying agent modifications...\n", target_file);
    if (!create_backup(target_file, backup_file)) {
        printf("ERROR: Failed to create backup file.\n");
        return 1;
    }

    apply_edit(target_file, agent_modified_code);

    // STEP C: Run test suite to verify the fix and check for regressions
    printf("\n[Step 3] Running regression tests on modified code...\n");
    if (execute_test_suite(test_cmd) == 0) {
        printf("\nSUCCESS: All tests passed! Changes accepted and backup cleaned.\n");
        remove(backup_file);
    } else {
        printf("\nREGRESSION DETECTED: Modified code broke tests!\n");
        printf("Initiating automatic rollback...\n");
        rollback_changes(target_file, backup_file);
        printf("Rollback complete: Code restored to original working state.\n");
    }

    return 0;
} 1)
