export default function log(){
  console.log('hello from package');
}
log();

npx webpack --config webpack.config.js

produces

(()=>{"use strict";console.log("hello from package")})();

which is surprising because i thought it'd preserve the export: what if it needs to be used in another module?




Webpack, by default, generates a bundle targeting the web, where ES6 import/export syntax isn't widely supported yet (as of my knowledge cutoff in September 2021). Instead, Webpack's default output library target is "var", which doesn't preserve module exports because it's intended to be loaded in a <script> tag, not imported as a module.

If you want to preserve ES6 module exports, you'll need to tell Webpack to output an ES6 module by setting output.libraryTarget to "module". However, this will only work in environments that support ES6 modules. Also note that as of Webpack 5, output.libraryTarget was replaced with output.library.type. Here's an updated config:

javascript
Copy code
const path = require('path');

module.exports = {
  entry: './src/script.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist'),
    library: {
      type: "module"
    }
  },
  experiments: {
    outputModule: true,
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /(node_modules)/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['@babel/preset-env']
          }
        }
      }
    ]
  }
};
Remember, this bundle.js must be loaded with type="module" in your HTML file:

html
Copy code
<script type="module" src="dist/bundle.js"></script>
Also, Webpack only includes modules in the bundle that are being used. If a module isn't imported by the entry porint (or by modules imported by the entry point), it won't be included in the bundle. If you want to use the exported function in another module, you'll have to import it in that module. The fact that it's not being included suggests that it's not being imported anywhere.