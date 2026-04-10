# HDRI Files

Drop `.hdr` equirectangular HDRI files here.

After adding files, update the `HDRI_FILES` array near the top of the `<script>` block in `index.html` so they appear in the Environment panel dropdown:

```js
const HDRI_FILES = [
  { label: 'Studio Soft',  file: 'studio_soft.hdr'  },
  { label: 'Sunset',       file: 'sunset.hdr'        },
  // add more here...
];
```

Good free sources:
- [Poly Haven](https://polyhaven.com/hdris) — CC0, download as .hdr
- [HDRIHaven](https://hdri-haven.com)
