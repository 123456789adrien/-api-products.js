const fs = require('fs');
const path = require('path');
const FILE = path.join('/tmp', 'fch-products.json');
let products = [];

function load() {
  try {
    if(fs.existsSync(FILE)) {
      products = JSON.parse(fs.readFileSync(FILE, 'utf8'));
    }
  } catch(e) { products = []; }
}

function save() {
  try { fs.writeFileSync(FILE, JSON.stringify(products)); } catch(e) {}
}

module.exports = async (req, res) => {
  res.setHeader('Access-Control-Allow-Origin', '*');
  res.setHeader('Access-Control-Allow-Methods', 'GET, POST, DELETE, OPTIONS');
  res.setHeader('Access-Control-Allow-Headers', 'Content-Type');
  
  if(req.method === 'OPTIONS') { res.status(204).end(); return; }
  
  load();
  
  if(req.method === 'GET') {
    res.status(200).json({ ok: true, products });
  } else if(req.method === 'POST') {
    const body = req.body || {};
    const product = {
      id: 'PRD_' + Date.now(),
      name: body.name || 'Product',
      category: body.category || 'Other',
      price: body.price || 0,
      description: body.description || '',
      created_at: new Date().toISOString()
    };
    products.push(product);
    save();
    res.status(200).json({ ok: true, product });
  } else if(req.method === 'DELETE') {
    const id = req.query.id;
    products = products.filter(p => p.id !== id);
    save();
    res.status(200).json({ ok: true });
  }
};
