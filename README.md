```js
import createBusyBox from "https://esm.sh/gh/MarketingPip/sed-wasm@796b7e1/es2022/dist/busybox.mjs"
async function runSed(text, pattern) {
    const instance = await createBusyBox();
    
    // Write your string to the virtual filesystem
    instance.FS.writeFile('/input.txt', text);
    
    let output = "";
    // Capture stdout
    instance.print = (data) => { output += data + "\n"; };

    // Run: busybox sed <pattern> /input.txt
    instance.callMain(['sed', pattern, '/input.txt']);
    
    return output.trim();
}

// Example usage
runSed("hello world", "s/hello/wasm/").then(console.log);

const instance = await createBusyBox({
    // Force the loader to use fetch even if it thinks it's in Node
    instantiateWasm: (info, receiveInstance) => {
        fetch('./dist/busybox.wasm')
            .then(response => response.arrayBuffer())
            .then(bytes => WebAssembly.instantiate(bytes, info))
            .then(result => receiveInstance(result.instance));
        return {}; // return empty object as required by Emscripten
    }
});
```
