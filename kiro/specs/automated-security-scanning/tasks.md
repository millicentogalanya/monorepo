# Implementation Plan: Automated Security Scanning

## Overview

This implementation plan breaks down the automated security scanning feature into actionable coding tasks. The system will integrate with GitHub Actions to provide comprehensive security analysis across frontend, backend, and smart contracts. The implementation follows a phased approach, building core infrastructure first, then integrating scanners, and finally adding reporting and error handling.

## Tasks

- [x] 1. Set up core infrastructure and data models
  - [x] 1.1 Create vulnerability schema TypeScript interfaces
    - Define `Vulnerability` interface with all required fields (id, source, severity, location, metadata, remediation)
    - Define `ScanResult` interface with timestamp, duration, summary, and vulnerabilities array
    - Define severity type union and source type union
    - _Requirements: 1.5, 2.4, 3.4, 6.2, 6.3_

  - [ ]\* 1.2 Write property test for vulnerability schema completeness
    - **Property 5: Complete Vulnerability Report Structure**
    - **Validates: Requirements 1.5, 2.4, 3.4**

  - [x] 1.3 Create result aggregator module
    - Implement function to collect outputs from multiple scanners
    - Implement deduplication logic for identical findings
    - Implement aggregate statistics calculation (total count, severity breakdown)
    - _Requirements: 4.2, 6.2_

  - [ ]\* 1.4 Write unit tests for result aggregator
    - Test deduplication of identical vulnerabilities
    - Test severity grouping and counting
    - Test handling of empty results
    - _Requirements: 4.2, 6.2_

- [x] 2. Implement dependency scanner integration
  - [x] 2.1 Create npm audit wrapper module
    - Implement function to execute npm audit with JSON output
    - Parse npm audit JSON output into vulnerability schema
    - Map npm audit fields to normalized schema (name → package, severity → severity, etc.)
    - Handle frontend and backend package-lock.json files
    - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5_

  - [ ]\* 2.2 Write property test for npm audit output parsing
    - **Property 4: Version-Specific Vulnerability Reporting**
    - **Validates: Requirements 1.4**

  - [x] 2.3 Create cargo audit wrapper module
    - Implement function to execute cargo audit with JSON output
    - Parse cargo audit JSON output into vulnerability schema
    - Map cargo audit fields to normalized schema (advisory.id → cve, package.name → package, etc.)
    - Handle contracts Cargo.lock file
    - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5_

  - [ ]\* 2.4 Write unit tests for dependency scanners
    - Test npm audit parsing with known vulnerable package (lodash@4.17.15)
    - Test cargo audit parsing
    - Test handling of packages with no vulnerabilities
    - Test error handling for missing lock files
    - _Requirements: 1.1, 1.2, 8.1_

- [x] 3. Checkpoint - Verify dependency scanning works
  - Ensure all tests pass, ask the user if questions arise.

- [x] 4. Implement static code analyzer integration
  - [x] 4.1 Configure ESLint with security plugins
    - Add eslint-plugin-security to dependencies
    - Create ESLint configuration with security rules enabled
    - Configure ESLint to output JSON format
    - _Requirements: 2.1, 2.2_

  - [x] 4.2 Create ESLint wrapper module
    - Implement function to execute ESLint on modified files
    - Parse ESLint JSON output into vulnerability schema
    - Map ESLint fields to normalized schema (filePath → file, line → line, ruleId → cwe)
    - Filter for security-related rules only
    - _Requirements: 2.1, 2.2, 2.3, 2.4, 2.5_

  - [ ]\* 4.3 Write unit tests for ESLint integration
    - Test detection of SQL injection patterns
    - Test detection of XSS vulnerabilities
    - Test handling of files with no issues
    - Test ESLint output parsing
    - _Requirements: 2.2, 8.3_

  - [x] 4.4 Configure Semgrep with OWASP ruleset
    - Create Semgrep configuration file with security rules
    - Add rules for SQL injection, XSS, insecure crypto, path traversal
    - Configure Semgrep to output JSON format
    - _Requirements: 2.1, 2.2_

  - [x] 4.5 Create Semgrep wrapper module
    - Implement function to execute Semgrep on modified files
    - Parse Semgrep JSON output into vulnerability schema
    - Map Semgrep fields to normalized schema
    - _Requirements: 2.1, 2.2, 2.3, 2.4, 2.5_

  - [ ]\* 4.6 Write property test for static analyzer
    - **Property 6: Security Issue Detection**
    - **Validates: Requirements 2.2**

  - [x] 4.7 Configure Clippy security lints for Rust
    - Add Clippy security lint configuration to contracts
    - Configure Clippy to output JSON format
    - _Requirements: 2.1, 2.2_

  - [x] 4.8 Create Clippy wrapper module
    - Implement function to execute Clippy on contract files
    - Parse Clippy JSON output into vulnerability schema
    - Map Clippy fields to normalized schema
    - _Requirements: 2.1, 2.2, 2.3, 2.4, 2.5_

- [x] 5. Checkpoint - Verify static analysis works
  - Ensure all tests pass, ask the user if questions arise.

- [x] 6. Implement secret scanner integration
  - [x] 6.1 Configure Gitleaks with custom patterns
    - Create Gitleaks configuration file extending default rules
    - Add custom patterns for API keys, database connection strings
    - Configure entropy thresholds
    - _Requirements: 3.1, 3.2, 3.5_

  - [x] 6.2 Create Gitleaks wrapper module
    - Implement function to execute Gitleaks on commit diffs
    - Parse Gitleaks JSON output into vulnerability schema
    - Map Gitleaks fields to normalized schema (File → file, StartLine → line, RuleID → title)
    - Ensure secret values are never included in output (redaction)
    - Set all detected secrets to severity 'critical'
    - _Requirements: 3.1, 3.2, 3.3, 3.4, 3.5_

  - [ ]\* 6.3 Write property test for secret scanner
    - **Property 7: Secret Type Detection**
    - **Validates: Requirements 3.2**

  - [ ]\* 6.4 Write unit tests for secret scanner
    - Test detection of AWS access keys
    - Test detection of GitHub tokens
    - Test detection of database connection strings
    - Test secret value redaction (verify secret not in output)
    - Test false positive handling (example keys in docs)
    - _Requirements: 3.2, 3.4, 8.2_

- [x] 7. Implement security scan orchestrator
  - [x] 7.1 Create orchestrator script (bash)
    - Implement main orchestration logic to coordinate all scanners
    - Execute dependency scanners (npm audit, cargo audit) in parallel
    - Execute static analyzers (ESLint, Semgrep, Clippy) in parallel
    - Execute secret scanner (Gitleaks)
    - Collect all results and pass to aggregator
    - _Requirements: 1.1, 2.1, 3.1, 4.5_

  - [ ]\* 7.2 Write property test for scanner execution
    - **Property 1: Scanner Execution on PR Events**
    - **Validates: Requirements 1.1, 2.1, 3.1**

  - [x] 7.3 Add timeout handling to orchestrator
    - Implement 2-minute timeout per scanner component
    - Implement 5-minute overall timeout for orchestrator
    - Handle timeout gracefully (mark as incomplete, continue with other scanners)
    - _Requirements: 4.5_

  - [x] 7.4 Add error handling for scanner failures
    - Implement try-catch for each scanner execution
    - Log errors but continue with other scanners
    - Generate partial results when some scanners fail
    - _Requirements: 4.5_

  - [ ]\* 7.5 Write unit tests for orchestrator error handling
    - Test timeout handling
    - Test partial results when scanners fail
    - Test parallel execution
    - _Requirements: 4.5_

- [x] 8. Checkpoint - Verify orchestrator coordinates all scanners
  - Ensure all tests pass, ask the user if questions arise.

- [x] 9. Implement report generator
  - [x] 9.1 Create JSON report generator
    - Implement function to generate machine-readable JSON report
    - Include scan timestamp, duration, scanned components, summary, and vulnerabilities
    - Validate report structure matches ScanResult interface
    - _Requirements: 6.1, 6.2, 6.3_

  - [ ]\* 9.2 Write property test for report generation
    - **Property 2: Vulnerability Report Generation**
    - **Validates: Requirements 1.2, 2.3, 3.3**

  - [x] 9.3 Create markdown report generator
    - Implement function to generate human-readable markdown report
    - Format vulnerabilities grouped by severity
    - Include summary statistics and detailed findings
    - Add links to references and remediation guidance
    - _Requirements: 6.1, 6.2, 6.3_

  - [ ]\* 9.4 Write property test for dual format reports
    - **Property 13: Dual Format Report Generation**
    - **Validates: Requirements 6.1**

  - [ ]\* 9.5 Write unit tests for report generator
    - Test markdown formatting
    - Test JSON structure validation
    - Test severity grouping
    - Test handling of empty results
    - _Requirements: 6.1, 6.2, 6.3_

- [x] 10. Implement GitHub Actions workflow
  - [x] 10.1 Create security-scan.yml workflow file
    - Define workflow triggers (pull_request: opened, synchronize, reopened)
    - Set required permissions (contents: read, pull-requests: write, checks: write, security-events: write)
    - Add checkout step with full history (fetch-depth: 0)
    - Add step to execute orchestrator script
    - Set 10-minute timeout for job
    - _Requirements: 1.1, 2.1, 3.1, 4.1, 4.5_

  - [x] 10.2 Add artifact upload step to workflow
    - Upload security-scan-results.json as artifact
    - Set 90-day retention period
    - Ensure upload runs even if scan fails (if: always())
    - _Requirements: 6.4, 6.5_

  - [ ]\* 10.3 Write property test for report artifact accessibility
    - **Property 15: Report Artifact Accessibility**
    - **Validates: Requirements 6.4**

- [x] 11. Implement GitHub Check Runs API integration
  - [x] 11.1 Create GitHub Check Run updater module
    - Implement function to create/update GitHub Check Run using actions/github-script
    - Set check run name to "Security Scan"
    - Include summary with vulnerability counts by severity
    - Add link to detailed report artifact
    - _Requirements: 4.1, 4.2, 4.3_

  - [ ]\* 11.2 Write property test for PR check updates
    - **Property 8: PR Check Update**
    - **Validates: Requirements 4.1**

  - [ ]\* 11.3 Write property test for severity breakdown display
    - **Property 9: PR Check Severity Breakdown**
    - **Validates: Requirements 4.2**

  - [ ]\* 11.4 Write property test for report link inclusion
    - **Property 10: PR Check Report Link**
    - **Validates: Requirements 4.3**

  - [x] 11.2 Implement status determination logic
    - Set status to 'failure' if any critical or high severity vulnerabilities found
    - Set status to 'success' with warnings if only medium/low vulnerabilities found
    - Set status to 'success' if no vulnerabilities found
    - Set status to 'neutral' if scanners fail
    - _Requirements: 4.4, 5.1, 5.2, 5.3, 5.4_

  - [ ]\* 11.6 Write property test for failing status on critical/high
    - **Property 11: Failing Status for Critical and High Vulnerabilities**
    - **Validates: Requirements 5.1, 5.2**

  - [ ]\* 11.7 Write property test for passing status on medium/low
    - **Property 12: Passing Status for Medium and Low Vulnerabilities**
    - **Validates: Requirements 5.4**

  - [x] 11.8 Add workflow step to fail job on critical/high vulnerabilities
    - Check orchestrator output status
    - Exit with code 1 if status is 'fail'
    - This blocks PR merge when critical/high vulnerabilities detected
    - _Requirements: 5.3_

  - [ ]\* 11.9 Write unit tests for status determination
    - Test status with various vulnerability combinations
    - Test merge blocking behavior
    - _Requirements: 5.1, 5.2, 5.3, 5.4_

- [x] 12. Checkpoint - Verify GitHub integration works
  - Ensure all tests pass, ask the user if questions arise.

- [x] 13. Implement PR comment notifications
  - [x] 13.1 Create PR comment poster module
    - Implement function to post comment on PR using actions/github-script
    - Include vulnerability count by severity in comment
    - Add link to full vulnerability report artifact
    - Format comment with markdown for readability
    - _Requirements: 7.1, 7.2, 7.3_

  - [ ]\* 13.2 Write property test for critical/high notifications
    - **Property 16: Critical/High Vulnerability Notifications**
    - **Validates: Requirements 7.1**

  - [ ]\* 13.3 Write property test for notification structure
    - **Property 17: Notification Comment Structure**
    - **Validates: Requirements 7.2**

  - [x] 13.4 Implement comment update logic (avoid duplicates)
    - Search for existing security scan comment on PR
    - Update existing comment if found
    - Create new comment only if no existing comment found
    - Use unique comment identifier (e.g., HTML comment marker)
    - _Requirements: 7.4_

  - [ ]\* 13.5 Write property test for idempotent comment updates
    - **Property 18: Idempotent Comment Updates**
    - **Validates: Requirements 7.4**

  - [x] 13.6 Add success comment for passing scans
    - Post success message when no vulnerabilities found
    - Keep message concise and positive
    - _Requirements: 7.3_

  - [ ]\* 13.7 Write unit tests for PR comment posting
    - Test comment formatting
    - Test comment update logic
    - Test success comment
    - Test handling of GitHub API rate limits
    - _Requirements: 7.1, 7.2, 7.3, 7.4_

- [x] 14. Implement error handling and resilience
  - [x] 14.1 Add retry logic for network failures
    - Implement exponential backoff for vulnerability database queries
    - Retry up to 3 times on network errors
    - Log all retry attempts
    - _Requirements: 1.3_

  - [x] 14.2 Add rate limit handling for GitHub API
    - Check rate limit before posting comments
    - If rate limited, store comment in artifact and log warning
    - Implement retry on next scan
    - _Requirements: 7.4_

  - [x] 14.3 Implement partial result handling
    - Generate report with available data when some scanners fail
    - Clearly indicate which scanners failed in report
    - Set PR check to warning status (not pass/fail) when scanners fail
    - _Requirements: 4.1, 6.2_

  - [x] 14.4 Add input validation and error handling
    - Handle malformed package-lock.json gracefully
    - Handle malformed Cargo.lock gracefully
    - Skip binary files in static analysis
    - Skip files > 1MB with warning
    - _Requirements: 2.1, 2.5_

  - [ ]\* 14.5 Write unit tests for error handling
    - Test retry logic
    - Test rate limit handling
    - Test partial results
    - Test invalid input handling
    - _Requirements: 1.3, 2.1, 4.1_

- [x] 15. Implement validation and test mode
  - [x] 15.1 Add test mode flag to orchestrator
    - Implement --test-mode command line flag
    - In test mode: run all scanners but don't update PR checks or block merges
    - In test mode: inject known test vulnerabilities
    - _Requirements: 8.5_

  - [x] 15.2 Create validation test suite
    - Test 1: Add lodash@4.17.15, verify CVE-2020-8203 detected
    - Test 2: Commit sample API key, verify detection
    - Test 3: Commit SQL injection pattern, verify detection
    - Test 4: Round-trip test (add vulnerability → remove → verify clean)
    - _Requirements: 8.1, 8.2, 8.3, 8.4_

  - [ ]\* 15.3 Write property test for round-trip validation
    - **Property 19: Vulnerability Detection Round-Trip**
    - **Validates: Requirements 8.4**

  - [ ]\* 15.4 Write integration tests for end-to-end workflow
    - Test complete workflow with test PR
    - Verify all scanners execute
    - Verify report generation
    - Verify PR check updates
    - Verify comment posting
    - Verify merge blocking
    - _Requirements: 1.1, 2.1, 3.1, 4.1, 5.3, 7.1_

- [x] 16. Final integration and wiring
  - [x] 16.1 Wire all components together in orchestrator
    - Connect scanner modules to orchestrator
    - Connect aggregator to report generator
    - Connect report generator to GitHub integration
    - Ensure proper error propagation
    - _Requirements: All_

  - [x] 16.2 Add comprehensive logging
    - Log scanner execution start/end times
    - Log vulnerability counts at each stage
    - Log errors with full context
    - Output logs to workflow console
    - _Requirements: 4.1, 6.2_

  - [x] 16.3 Create scanner tool configuration files
    - Create .gitleaks.toml with custom patterns
    - Create semgrep.yml with OWASP rules
    - Create .eslintrc.security.js with security plugins
    - Document configuration options
    - _Requirements: 2.1, 2.2, 3.1, 3.2, 3.5_

  - [x] 16.4 Update existing CI workflows to include security scan
    - Ensure security-scan.yml is triggered on all PRs
    - Add security scan as required check for merge
    - Document workflow in repository README
    - _Requirements: 4.1, 5.3_

- [x] 17. Final checkpoint - Run complete validation suite
  - Ensure all tests pass, ask the user if questions arise.

## Notes

- Tasks marked with `*` are optional and can be skipped for faster MVP
- Each task references specific requirements for traceability
- Property tests validate universal correctness properties from the design document
- Unit tests validate specific examples and edge cases
- The implementation uses TypeScript for core logic and bash for orchestration
- All scanner tools (npm audit, cargo audit, Gitleaks, ESLint, Semgrep, Clippy) are industry-standard open-source tools
- The system integrates with GitHub Actions and uses the GitHub API for PR checks and comments
- Error handling ensures the system fails gracefully and doesn't block merges on scanner failures
- Test mode allows validation without affecting PR status
