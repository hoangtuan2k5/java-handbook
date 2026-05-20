# Toolchain and Quality Gates

## Why this file exists

Toolchain là bộ công cụ Java project dùng để compile, format, lint/static analysis, test, measure coverage, package, và verify code. Quality gate là tập command bắt buộc phải pass trước khi merge, release, hoặc xem một change là xong.

Nếu chỉ nói code “nên giống Java” mà không nói build/test/check command nào phải pass, convention vẫn thiếu phần vận hành quan trọng nhất.

## Rules

- Java project nghiêm túc phải có một build tool rõ: Maven hoặc Gradle.
- Local verification và CI verification nên chạy cùng lifecycle/task chính.
- Không rely vào IDE-only build; command line build phải pass.
- Unit tests nên chạy trong default verification; integration tests có thể tách profile/task riêng nếu chậm hoặc cần external dependency.
- Formatter/static analysis nên được config trong repo, không phụ thuộc máy cá nhân.
- Coverage threshold nếu được dùng làm gate phải ổn định và không hạ tùy tiện để merge.
- Library nên verify trên JDK versions được support; service nên verify trên JDK version deploy thật.

## Toolchain decision matrix

| Need | Maven | Gradle | Avoid |
|---|---|---|---|
| Compile + unit tests | `mvn test` | `./gradlew test` | chỉ chạy test từ IDE |
| Full local verification | `mvn verify` | `./gradlew check` | command không rõ trong README |
| Formatting | Spotless hoặc formatter plugin | Spotless | style thủ công mỗi người một kiểu |
| Static analysis | Checkstyle, PMD, SpotBugs | Checkstyle, PMD, SpotBugs | analyzer chỉ chạy optional không ai biết |
| Coverage | JaCoCo | JaCoCo | coverage report không có threshold khi cần gate |
| Multi-JDK library | Maven profiles/toolchains/CI matrix | Gradle toolchains/CI matrix | claim support nhiều JDK nhưng chỉ test một version |

## Minimum gates by project style

| Project style | Minimum gate | Extra gate when project grows |
|---|---|---|
| Learning/exercise project | compile + unit tests | Checkstyle/formatter nếu nhiều người sửa |
| Library | compile, unit tests, static analysis, coverage, package artifact | multi-JDK matrix, javadoc jar/source jar checks |
| Spring/service | compile, unit tests, integration tests, package image/jar | container smoke test, dependency vulnerability scan |
| CLI/batch job | compile, unit tests, package runnable artifact | end-to-end sample input test |

## Maven and Gradle lifecycle map

| Intent | Maven | Gradle | Notes |
|---|---|---|---|
| Compile only | `mvn compile` | `./gradlew classes` | fast syntax/type feedback |
| Unit tests | `mvn test` | `./gradlew test` | should run by default during normal verification |
| Integration tests | `mvn verify` with Failsafe | custom `integrationTest` task or plugin config | keep slow/external tests separate from unit tests |
| Full verification | `mvn verify` | `./gradlew check` | best default CI gate for most projects |
| Build artifact | `mvn package` | `./gradlew build` | creates jar/war/image-related artifacts depending on config |
| Local install | `mvn install` | usually not needed; publish to local repo only when required | do not use as default gate if `verify` is enough |

## Unit vs integration test boundary

- Maven unit tests usually run through Surefire in the `test` phase.
- Maven integration tests usually run through Failsafe in `integration-test`/`verify`.
- Gradle unit tests usually run in `test`; integration tests should have a separate source set/task when they are slower or need external dependencies.
- Unit tests should not need real network, database, or container by default.
- Integration tests should be explicit, reproducible, and documented as part of the quality gate.

## Security and dependency gates

| Need | Common gate | Why |
|---|---|---|
| Vulnerable dependency scan | OWASP Dependency-Check, Snyk, GitHub Dependabot/CodeQL depending on project | catches known CVEs before release |
| Dependency convergence | Maven Enforcer or Gradle dependency constraints | avoids classpath surprises |
| Bytecode/static bug scan | SpotBugs | catches common bug patterns beyond compiler checks |
| Style/API drift | Checkstyle/PMD/Error Prone if team uses them | keeps codebase consistent and catches risky patterns |

## Local vs CI gates

| Gate | Local default | CI default |
|---|---|---|
| Fast compile/test | `mvn test` or `./gradlew test` | same command or included in full gate |
| Full verification | optional before small commits | `mvn verify` or `./gradlew check` required |
| Integration tests | run before pushing risky changes | required on main/PR if dependencies are available |
| Coverage/security scan | optional local | required for release or protected branches |
| Multi-JDK matrix | rarely local | required for libraries that claim multiple JDK support |

## Good examples

```bash
mvn -q verify
```

```bash
./gradlew clean check
```

```bash
./gradlew test integrationTest check
```

```bash
mvn -q -DskipTests compile
mvn -q test
mvn -q verify
```

## Bad examples

- README chỉ nói “run tests in IDE”.
- CI chỉ compile nhưng không chạy tests.
- Project dùng formatter trong IDE nhưng không có config trong repo.
- Static analysis có config nhưng không được bind vào `verify` hoặc `check`.
- Integration tests luôn chạy trong unit-test phase và làm local feedback quá chậm.

## Exceptions

- Exercise skeleton có thể chỉ yêu cầu compile nếu test suite chưa tồn tại, nhưng phải nói rõ command compile chuẩn.
- Prototype nhỏ có thể chưa cần Checkstyle/SpotBugs/coverage, nhưng vẫn cần command line compile/test.
- Framework hoặc company build pipeline có thể dùng tool khác; rule quan trọng là gate rõ, reproducible, và CI enforce được.
- Generated code nên exclude khỏi formatter/static analysis nếu tool không hiểu hoặc code không thuộc ownership của project.

## Notes

Với Java, quality gate tối thiểu gần như luôn bắt đầu từ compile. Type safety mạnh nhưng không thay thế tests, static analysis, hay integration checks.

Một gate tốt nên tách fast feedback và slow verification: unit tests chạy thường xuyên; integration tests, coverage, security scan, và package verification có thể chạy trong CI hoặc profile riêng.

## Official references

- [Maven: Build Lifecycle](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html)
- [Gradle: Build Lifecycle](https://docs.gradle.org/current/userguide/build_lifecycle.html)
- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [JaCoCo](https://www.jacoco.org/jacoco/)
- [Checkstyle](https://checkstyle.org/)
- [SpotBugs](https://spotbugs.github.io/)
- [PMD](https://pmd.github.io/)
- [Spotless](https://github.com/diffplug/spotless)

## Related rules

[[012-imports-formatting-and-order]]

[[013-exception-handling-guidelines]]

[[015-javadoc-and-comments]]

[[016-common-java-anti-patterns]]
