# Project Setup Guide

**Note:** You can ask an AI assistant for help with setup if you run into any issues or need platform-specific instructions.

---

## TypeScript/Vite

### Initial Setup
```bash
npm create vite@latest my-project-name -- --template vanilla-ts
cd my-project-name
npm install
npm run dev
```

### What This Does

- Creates a new Vite project with TypeScript template
- Installs all dependencies
- Starts the development server (usually at http://localhost:5173)

### Common Issues

- If `npm create vite` fails, try `npx create-vite@latest`
- If port 5173 is in use, Vite will automatically try the next available port

---

## Go/Gin

### Initial Setup
```bash
mkdir my-project-name
cd my-project-name
go mod init github.com/yourusername/my-project-name
go get -u github.com/gin-gonic/gin
```

### Create main.go
```go
package main

import "github.com/gin-gonic/gin"

func main() {
    r := gin.Default()
    
    r.GET("/ping", func(c *gin.Context) {
        c.JSON(200, gin.H{"message": "pong"})
    })
    
    r.Run(":8080") // Listen on localhost:8080
}
```

### Run the Server
```bash
go run main.go
```

### What This Does

- Creates a basic Gin web server
- Sets up a `/ping` endpoint that responds with JSON
- Server runs on http://localhost:8080

### Common Issues

- If port 8080 is in use, change `:8080` to another port like `:3000`
- Make sure Go is installed: `go version`

---

## Go/Gin with GORM & PostgreSQL

### Install PostgreSQL

**macOS:**
```bash
brew install postgresql@15
brew services start postgresql@15
```

**Linux (Ubuntu/Debian):**
```bash
sudo apt-get update
sudo apt-get install postgresql postgresql-contrib
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

**Windows:**
- Download installer from https://www.postgresql.org/download/windows/
- Follow installation wizard
- Remember the password you set for the postgres user

### Create a Database

**Option 1 - Command line:**
```bash
createdb mydb
```

**Option 2 - Using psql:**
```bash
psql postgres
```
Then in psql:
```sql
CREATE DATABASE mydb;
\q
```

**For Linux, you may need to switch to postgres user first:**
```bash
sudo -u postgres psql
CREATE DATABASE mydb;
\q
```

### Install Go Dependencies
```bash
go get -u gorm.io/gorm
go get -u gorm.io/driver/postgres
```

### Example Connection in main.go
```go
package main

import (
    "fmt"
    "gorm.io/driver/postgres"
    "gorm.io/gorm"
)

func main() {
    // Connection string format:
    // host=localhost user=username password=pass dbname=mydb port=5432 sslmode=disable
    
    // Default PostgreSQL username is usually 'postgres'
    // Use the password you set during PostgreSQL installation
    dsn := "host=localhost user=postgres password=yourpassword dbname=mydb port=5432 sslmode=disable"
    
    db, err := gorm.Open(postgres.Open(dsn), &gorm.Config{})
    if err != nil {
        panic("failed to connect to database: " + err.Error())
    }
    
    fmt.Println("Database connection successful!")
    
    // Use db here for your queries
    // Example: db.AutoMigrate(&YourModel{})
}
```

### Connection String Parameters

- `host`: Database server address (usually `localhost`)
- `user`: PostgreSQL username (default is often `postgres`)
- `password`: Password for the PostgreSQL user
- `dbname`: Name of the database you created
- `port`: PostgreSQL port (default is `5432`)
- `sslmode`: Use `disable` for local development

### Common Issues

- **Authentication failed**: Make sure your username and password are correct
- **Database does not exist**: Create the database first using `createdb mydb`
- **Connection refused**: PostgreSQL service might not be running
  - macOS: `brew services start postgresql@15`
  - Linux: `sudo systemctl start postgresql`
  - Windows: Start PostgreSQL service from Services app
- **Peer authentication failed** (Linux): Edit `/etc/postgresql/[version]/main/pg_hba.conf` and change `peer` to `md5` for local connections, then restart PostgreSQL

### Testing the Connection

Run your program:
```bash
go run main.go
```

You should see: `Database connection successful!`

---

## Flutter

### Prerequisites

Install Flutter SDK first: https://docs.flutter.dev/get-started/install

**Verify installation:**
```bash
flutter doctor
```

This command checks for any missing dependencies or issues with your Flutter setup.

### Initial Setup
```bash
flutter create my_project_name
cd my_project_name
flutter run
```

### What This Does

- Creates a new Flutter project with sample counter app
- `flutter run` launches the app on your connected device or emulator

### Platform-Specific Setup

**Android:**
- Install Android Studio
- Set up an Android emulator or connect a physical device
- Enable USB debugging on physical device

**iOS (macOS only):**
- Install Xcode from Mac App Store
- Open Xcode once to install additional components
- Set up iOS Simulator or connect a physical device

**Web:**
```bash
flutter run -d chrome
```

**Desktop:**
```bash
# macOS
flutter run -d macos

# Windows
flutter run -d windows

# Linux
flutter run -d linux
```

### Common Issues

- **No devices found**: Make sure an emulator is running or a device is connected
- **Android licenses not accepted**: Run `flutter doctor --android-licenses` and accept all
- **iOS build fails**: Open `ios/Runner.xcworkspace` in Xcode and build from there to see detailed errors

### Useful Commands
```bash
# List all connected devices
flutter devices

# Run on specific device
flutter run -d <device-id>

# Build for release
flutter build apk        # Android
flutter build ios        # iOS
flutter build web        # Web

# Clean build cache (fixes many issues)
flutter clean
```

---

## General Tips

1. **Always check versions**: Run `node --version`, `go version`, or `flutter --version` to verify installations
2. **Read error messages carefully**: They often contain the solution
3. **Check firewall/antivirus**: These can block development servers
4. **Use version control**: Initialize git in your project (`git init`) to track changes
5. **Environment variables**: Store sensitive data (like database passwords) in `.env` files, not in code
