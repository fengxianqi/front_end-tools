## cliboard.js
```javascript

function _fallbackCopyTextToClipboard (text) {
  var textArea = document.createElement('textarea')
  textArea.value = text
  document.body.appendChild(textArea)
  textArea.focus()
  textArea.select()

  const promise = new Promise((resolve, reject) => {
    try {
      const successful = document.execCommand('copy')
      resolve(successful)
    } catch (err) {
      reject(err)
    }
  })

  document.body.removeChild(textArea)
  return promise
}

function copyTextToClipboard (text) {
  if (!navigator.clipboard) {
    return _fallbackCopyTextToClipboard(text)
  }
  return navigator.clipboard.writeText(text).then(() => {
    return true
  }).catch((err) => {
    throw err
  })
}

export default copyTextToClipboard

```
### usage
```
import copyToClipboard from './clipboard';
copyToClipboard('test string').then(() => {
  console.log('copy success!')
})
```


## cliboard.ts
```typescript
function _fallbackCopyTextToClipboard (text: string): Promise<boolean> {
  const textArea = document.createElement('textarea');
  textArea.value = text;
  document.body.appendChild(textArea);
  textArea.focus();
  textArea.select();

  const promise: Promise<boolean> = new Promise((resolve, reject) => {
    try {
      const successful = document.execCommand('copy');
      resolve(successful);
    } catch (err) {
      reject(err);
    }
  });

  document.body.removeChild(textArea);
  return promise;
}

function copyTextToClipboard (text: string): Promise<boolean> {
  if (!navigator.clipboard) {
    return _fallbackCopyTextToClipboard(text);
  }
  return navigator.clipboard.writeText(text).then(() => {
    return true;
  }).catch((err) => {
    throw err;
  });
}

export default copyTextToClipboard;

```
