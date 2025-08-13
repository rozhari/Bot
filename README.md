
const { execSync } = require('child_process');
const fs = require('fs');
const path = require('path');

const SESSION = "RGNK~aEaAPDZ1";

function run(command, options = {}) {
  try {
    execSync(command, { stdio: 'inherit', ...options });
  } catch (error) {
    console.error('❌ Error running command: ' + command);
    process.exit(1);
  }
}

// 1. Check FFmpeg
try {
  execSync('ffmpeg -version', { stdio: 'ignore' });
  console.log("⚡ System FFmpeg found, skipping download.");
} catch {
  if (!fs.existsSync('./ffmpeg')) {
    console.log("🔧 Downloading FFmpeg...");
    run('curl -L https://johnvansickle.com/ffmpeg/releases/ffmpeg-release-amd64-static.tar.xz -o ffmpeg.tar.xz');
    run('tar -xf ffmpeg.tar.xz');
    const ffmpegDir = fs.readdirSync('.').find(d => /^ffmpeg-.*-static$/.test(d));
    if (!ffmpegDir) {
      console.error('FFmpeg static directory not found after extraction.');
      process.exit(1);
    }
    fs.renameSync(path.join(ffmpegDir, 'ffmpeg'), './ffmpeg');
    run('chmod +x ./ffmpeg');
    run(`rm -rf ffmpeg.tar.xz ${ffmpegDir}`);
    console.log("✅ FFmpeg ready.");
  } else {
    console.log("⚡ Local FFmpeg binary already exists.");
  }
}

// 2. Clone raganork-md
if (!fs.existsSync('./raganork-md')) {
  console.log("📥 Cloning raganork-md (shallow)...");
  run('git clone --depth=1 https://github.com/souravkl11/raganork-md');
} else {
  console.log("🔄 raganork-md already cloned.");
}

try {
  process.chdir('./raganork-md');
} catch {
  console.error('❌ Failed to change directory to raganork-md!');
  process.exit(1);
}

// 3. Install Yarn if not present
try {
  execSync('yarn --version', { stdio: 'ignore' });
} catch {
  console.log("📦 Installing yarn with Corepack...");
  run('corepack enable');
  run('corepack prepare yarn@1.22.22 --activate');
}

// 4. Install dependencies only if node_modules missing
if (!fs.existsSync('node_modules')) {
  console.log("📦 Installing dependencies with yarn (cached, parallel)...");
  run('yarn install --ignore-engines --prefer-offline --network-concurrency 8');
} else {
  console.log("⚡ Dependencies already installed.");
}

// 5. Write session key
console.log("🔐 Writing session...");
fs.writeFileSync('config.env', 'SESSION=' + SESSION + '\n');

// 6. Start bot
console.log("🚀 Starting bot...");
run('yarn start');# Bot
