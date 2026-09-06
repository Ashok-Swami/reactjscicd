# React + Vite CI/CD Project

A modern React application built with Vite, featuring automated CI/CD pipelines for seamless development and deployment.

## 📋 Project Overview

This project demonstrates best practices for setting up a React application with:
- **Vite** - Lightning-fast frontend build tool with Hot Module Replacement (HMR)
- **React** - UI library for building interactive user interfaces
- **CI/CD Automation** - Automated testing and deployment pipelines
- **Docker** - Containerization for consistent deployment across environments

## 🛠️ Tech Stack

- **Frontend**: React + Vite
- **Styling**: CSS
- **Markup**: HTML
- **Deployment**: Docker

## 🚀 Getting Started

### Prerequisites
- Node.js (v16 or higher)
- npm or yarn package manager

### Installation

1. **Clone the repository**
   ```bash
   git clone https://github.com/Ashok-Swami/reactjscicd.git
   cd reactjscicd
   ```

2. **Install dependencies**
   ```bash
   npm install
   ```

3. **Start the development server**
   ```bash
   npm run dev
   ```
   The application will be available at `http://localhost:5173`

## 📦 Build & Deployment

### Build for Production
```bash
npm run build
```
This creates an optimized production build in the `dist/` directory.

### Docker Deployment
```bash
docker build -t reactjscicd .
docker run -p 80:3000 reactjscicd
```

## 🔧 Available Scripts

- `npm run dev` - Start development server with HMR
- `npm run build` - Create production build
- `npm run preview` - Preview production build locally
- `npm run lint` - Run ESLint to check code quality

## 🔌 Vite Plugins

This project supports two React plugins:

- **[@vitejs/plugin-react](https://github.com/vitejs/vite-plugin-react)** - Uses [Oxc](https://oxc.rs) for fast compilation
- **[@vitejs/plugin-react-swc](https://github.com/vitejs/vite-plugin-react)** - Uses [SWC](https://swc.rs/) for even faster builds

## 📊 Project Statistics

- **CSS**: 49.4%
- **JavaScript**: 44.9%
- **HTML**: 3.5%
- **Dockerfile**: 2.2%

## 📝 Learn More

- [Vite Documentation](https://vitejs.dev)
- [React Documentation](https://react.dev)
- [ESLint Configuration Guide](https://github.com/vitejs/vite/tree/main/packages/create-vite/template-react)

## 🤝 Contributing

Feel free to submit issues and pull requests to help improve this project!

## 📄 License

This project is open source and available under the MIT License.

---

**Happy Coding! 🎉**
