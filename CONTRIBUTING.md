# Contributing to homebrew-clawrtc

Thank you for your interest in contributing to the homebrew-clawrtc project! This document provides guidelines and instructions for contributing to this Homebrew tap for ClawRTC.

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [Getting Started](#getting-started)
- [How to Contribute](#how-to-contribute)
- [Development Setup](#development-setup)
- [Formula Guidelines](#formula-guidelines)
- [Testing](#testing)
- [Submitting Changes](#submitting-changes)
- [Resources](#resources)

## Code of Conduct

This project adheres to a code of conduct that we expect all contributors to follow:

- Be respectful and inclusive in all interactions
- Welcome newcomers and help them learn
- Focus on constructive feedback
- Respect differing viewpoints and experiences

## Getting Started

### Prerequisites

Before contributing, ensure you have:

- **macOS** (10.14+) or **Linux** (Ubuntu 18.04+, Debian 9+, Fedora 30+)
- **Homebrew** installed (for macOS/Linux)
- **Git** (2.20+)
- **Ruby** (2.6+) - for Homebrew development
- **Xcode Command Line Tools** (macOS only)

### Quick Start

1. **Fork the repository** on GitHub
2. **Clone your fork**:
   ```bash
   git clone https://github.com/YOUR_USERNAME/homebrew-clawrtc.git
   cd homebrew-clawrtc
   ```
3. **Add the upstream remote**:
   ```bash
   git remote add upstream https://github.com/Scottcjn/homebrew-clawrtc.git
   ```

## How to Contribute

### Types of Contributions

We welcome the following types of contributions:

- **New formulas** for ClawRTC-related tools
- **Formula updates** (version bumps, dependency updates)
- **Bug fixes** for existing formulas
- **Documentation improvements**
- **Test improvements**
- **CI/CD enhancements**

### Reporting Issues

When reporting issues, please include:

- **Homebrew version**: `brew --version`
- **macOS/Linux version**: `sw_vers` (macOS) or `lsb_release -a` (Linux)
- **Command used**: The exact `brew` command that failed
- **Error output**: Full error message with `--verbose` flag
- **Steps to reproduce**: Minimal steps to trigger the issue

Example:
```bash
# Report template
brew --version
brew install --verbose --debug scottcjn/clawrtc/clawrtc 2>&1
```

## Development Setup

### Setting Up Homebrew Development Environment

1. **Install Homebrew** (if not already installed):
   ```bash
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```

2. **Install development dependencies**:
   ```bash
   brew install ruby shellcheck
   ```

3. **Set up your tap locally**:
   ```bash
   # Create a symlink to your development tap
   ln -s $(pwd) $(brew --repo)/Library/Taps/scottcjn/homebrew-clawrtc
   
   # Or use the tap command
   brew tap scottcjn/clawrtc
   brew tap --force-auto-update scottcjn/clawrtc $(pwd)
   ```

### Understanding the Repository Structure

```
homebrew-clawrtc/
├── Formula/
│   └── clawrtc.rb          # Main ClawRTC formula
├── README.md               # Project documentation
├── LICENSE                 # License file
└── .github/
    └── workflows/          # CI/CD workflows
```

## Formula Guidelines

### Formula Structure

A typical ClawRTC formula follows this structure:

```ruby
class Clawrtc < Formula
  desc "Mine RTC tokens with Proof of Antiquity consensus"
  homepage "https://github.com/Scottcjn/clawrtc"
  url "https://github.com/Scottcjn/clawrtc/archive/v1.0.0.tar.gz"
  sha256 "abc123..."
  license "MIT"

  depends_on "rust" => :build
  depends_on "openssl"

  def install
    system "cargo", "build", "--release"
    bin.install "target/release/clawrtc"
  end

  test do
    assert_match version.to_s, shell_output("#{bin}/clawrtc --version")
  end
end
```

### Writing Formulas

#### 1. Formula Naming

- Use lowercase: `clawrtc`, not `ClawRTC`
- Match the upstream project name
- For versioned formulas: `clawrtc@1.0`

#### 2. URL and SHA256

Always verify the SHA256 checksum:

```bash
# Download and calculate SHA256
curl -L -o clawrtc.tar.gz https://github.com/.../v1.0.0.tar.gz
shasum -a 256 clawrtc.tar.gz

# Or use Homebrew's built-in helper
brew fetch --build-from-source ./Formula/clawrtc.rb
```

#### 3. Dependencies

Specify dependencies accurately:

```ruby
# Build dependencies (not needed at runtime)
depends_on "rust" => :build
depends_on "cmake" => :build

# Runtime dependencies
depends_on "openssl"
depends_on "libuv"

# Optional dependencies
depends_on "cuda" => :optional
```

#### 4. Installation Methods

Common installation patterns:

**Rust projects**:
```ruby
def install
  system "cargo", "install", *std_cargo_args
end
```

**Make-based projects**:
```ruby
def install
  system "./configure", "--prefix=#{prefix}"
  system "make"
  system "make", "install"
end
```

**CMake projects**:
```ruby
def install
  system "cmake", "-S", ".", "-B", "build", *std_cmake_args
  system "cmake", "--build", "build"
  system "cmake", "--install", "build"
end
```

### ClawRTC-Specific Considerations

ClawRTC is a Rust-based cryptocurrency miner with specific requirements:

1. **Rust toolchain**: Ensure `depends_on "rust" => :build`
2. **OpenSSL**: Required for cryptographic operations
3. **Network access**: Tests should not require live network
4. **Configuration**: Default config should work out-of-the-box

Example ClawRTC formula:

```ruby
class Clawrtc < Formula
  desc "Mine RTC tokens with Proof of Antiquity consensus"
  homepage "https://github.com/Scottcjn/clawrtc"
  url "https://github.com/Scottcjn/clawrtc/archive/v1.0.0.tar.gz"
  sha256 "YOUR_SHA256_HERE"
  license "MIT"

  depends_on "rust" => :build
  depends_on "openssl"

  def install
    system "cargo", "install", *std_cargo_args
  end

  test do
    # Test version output
    assert_match version.to_s, shell_output("#{bin}/clawrtc --version")
    
    # Test config generation (no network)
    system "#{bin}/clawrtc", "--init-config"
    assert_predicate testpath/".clawrtc/config.toml", :exist?
  end
end
```

## Testing

### Local Testing

Test your formula locally before submitting:

```bash
# Install from local formula
brew install --build-from-source ./Formula/clawrtc.rb

# Run tests
brew test ./Formula/clawrtc.rb

# Audit formula (check for issues)
brew audit --new-formula ./Formula/clawrtc.rb

# Full check
brew audit --strict --online ./Formula/clawrtc.rb
```

### Common Issues and Fixes

| Issue | Cause | Fix |
|-------|-------|-----|
| `SHA256 mismatch` | Downloaded file changed | Update SHA256 |
| `Missing dependency` | Runtime library not declared | Add `depends_on` |
| `Test failure` | Network required | Mock responses or skip network tests |
| `Audit failure` | Style/format issue | Run `brew style --fix` |

### Continuous Integration

This repository uses GitHub Actions for CI:

- **macOS builds**: Tests on macOS 11, 12, 13
- **Linux builds**: Tests on Ubuntu 20.04, 22.04
- **Formula audit**: Automatic `brew audit` checks
- **Bottle building**: Automatic binary package creation

## Submitting Changes

### Pull Request Process

1. **Create a branch**:
   ```bash
   git checkout -b feature/my-contribution
   ```

2. **Make your changes** following the guidelines above

3. **Test locally**:
   ```bash
   brew audit --strict ./Formula/clawrtc.rb
   brew test ./Formula/clawrtc.rb
   ```

4. **Commit with a clear message**:
   ```bash
   git commit -m "feat: add clawrtc formula v1.0.0"
   ```

5. **Push to your fork**:
   ```bash
   git push origin feature/my-contribution
   ```

6. **Create a Pull Request** on GitHub

### Commit Message Format

Follow conventional commits:

- `feat: add new formula for X`
- `