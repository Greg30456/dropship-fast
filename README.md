package.json
{
  "name": "dropship-fast",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "dev": "next dev -p 3000",
    "build": "next build",
    "start": "next start"
  },
  "dependencies": {
    "axios": "^1.4.0",
    "next": "13.5.0",
    "react": "18.2.0",
    "react-dom": "18.2.0",
    "swr": "^2.2.0",
    "stripe": "^12.11.0"
  }
}
 
next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = { reactStrictMode: true }
module.exports = nextConfig
 
.env.example
# Fill these into .env.local for development or in Vercel env vars for production
NEXT_PUBLIC_BASE_URL=http://localhost:3000

# Stripe
STRIPE_SECRET_KEY=sk_test_xxx
STRIPE_WEBHOOK_SECRET=whsec_xxx

# CJdropshipping (demo)
CJ_CLIENT_ID=your_cj_client_id
CJ_CLIENT_SECRET=your_cj_client_secret
CJ_BASE_URL=https://developers.cjdropshipping.com/api

# EasyPost
EASYPOST_API_KEY=epk_test_xxx

# Email (SendGrid / Resend)
EMAIL_API_KEY=your_email_api_key
EMAIL_FROM=contact@yoursite.com

# Business
BUSINESS_NAME=Ton Entreprise
BUSINESS_VAT_NUMBER=FRXX999999999

# Security
JWT_SECRET=some_random_secret
 
README.md
(Contenu abrégé — voir le README complet dans le document créé précédemment)
# Dropship-Fast
Demo Next.js dropshipping starter optimisé pour livraison rapide en France.

## Installation
1. Copier les fichiers
2. `npm install`
3. `cp .env.example .env.local` puis remplir les clés
4. `npm run dev`

## Déploiement
- Pousser sur GitHub
- Importer le repo dans Vercel
- Configurer les variables d'environnement dans Vercel
- Déployer
 
styles/globals.css
html,body,#__next{height:100%}
body{font-family:Inter, ui-sans-serif, system-ui, -apple-system, 'Segoe UI', Roboto, 'Helvetica Neue', Arial; margin:0; background:#f7fafc}
img{max-width:100%;}
.container{max-width:1100px;margin:0 auto;padding:20px}
 
pages/_app.jsx
import '../styles/globals.css'
export default function App({ Component, pageProps }) {
  return <Component {...pageProps} />
}
 
pages/index.jsx
import useSWR from 'swr'
import axios from 'axios'
import ProductCard from '../components/ProductCard'

const fetcher = (url) => axios.get(url).then(r => r.data)

export default function Home() {
  const { data, error } = useSWR('/api/suppliers', fetcher)
  if (error) return <div className="container">Erreur de chargement</div>
  if (!data) return <div className="container">Chargement...</div>

  return (
    <div className="container">
      <h1>Produits — Dropship‑Fast (démo)</h1>
      <div style={{display:'grid',gridTemplateColumns:'repeat(auto-fit,minmax(260px,1fr))',gap:16,marginTop:16}}>
        {data.products.map(p => <ProductCard key={p.id} product={p} />)}
      </div>
    </div>
  )
}
 
components/ProductCard.jsx
import Link from 'next/link'
export default function ProductCard({ product }) {
  return (
    <article style={{background:'#fff',padding:12,borderRadius:8,boxShadow:'0 1px 4px rgba(0,0,0,0.08)'}}>
      <img src={product.img} alt={product.title} style={{height:190,objectFit:'cover',borderRadius:6}} />
      <h3 style={{marginTop:10}}>{product.title}</h3>
      <p style={{color:'#555'}}>{product.price} €</p>
      <div style={{marginTop:8}}>
        <Link href={`/product/${product.id}`}><a style={{textDecoration:'underline'}}>Voir le produit</a></Link>
      </div>
    </article>
  )
}
 
pages/product/[id].jsx
import { useRouter } from 'next/router'
import useSWR from 'swr'
import axios from 'axios'

const fetcher = (url) => axios.get(url).then(r => r.data)

export default function ProductPage() {
  const router = useRouter()
  const { id } = router.query
  const { data } = useSWR('/api/suppliers', fetcher)
  if (!data) return <div className="container">Chargement...</div>
  const product = data.products.find(p => p.id === id)
  if (!product) return <div className="container">Produit introuvable</div>

  return (
    <div className="container">
      <img src={product.img} alt={product.title} style={{width:'100%',height:360,objectFit:'cover',borderRadius:8}} />
      <h1 style={{marginTop:12}}>{product.title}</h1>
      <p style={{color:'#333'}}>{product.price} €</p>
      <h3>Fournisseurs (priorité EU pour livraison rapide)</h3>
      <ul>
        {product.suppliers.map(s => (
          <li key={s.id}><a href={s.link} target="_blank" rel="noreferrer">{s.name}</a> — {s.country} — délai indicatif: {s.leadDays}j</li>
        ))}
      </ul>
      <div style={{marginTop:12}}>
        <button onClick={() => alert('Simulation: ajout au panier')} style={{padding:'10px 16px',background:'#0066ff',color:'#fff',border:'none',borderRadius:6}}>Ajouter au panier</button>
      </div>
    </div>
  )
}
 
pages/cart.jsx
import { useState } from 'react'

export default function Cart() {
  const [customer, setCustomer] = useState({ name: '', email: '', country: 'FR', address: '' })
  const [loading, setLoading] = useState(false)

  async function placeOrder() {
    setLoading(true)
    const cart = JSON.parse(localStorage.getItem('cart') || '[]')
    const res = await fetch('/api/order', { method: 'POST', headers: {'Content-Type':'application/json'}, body: JSON.stringify({ cart, customer }) })
    const data = await res.json()
    alert('Commande simulée: ' + JSON.stringify(data, null, 2))
    setLoading(false)
  }

  return (
    <div className="container">
      <h1>Panier (démo)</h1>
      <div>
        <label>Nom<input value={customer.name} onChange={e => setCustomer({...customer,name:e.target.value})} /></label>
        <label>Email<input value={customer.email} onChange={e => setCustomer({...customer,email:e.target.value})} /></label>
        <label>Pays<select value={customer.country} onChange={e => setCustomer({...customer,country:e.target.value})}><option value="FR">France</option><option value="DE">Allemagne</option></select></label>
        <div style={{marginTop:12}}>
          <button onClick={placeOrder} disabled={loading} style={{padding:'10px 14px',background:'#16a34a',color:'#fff',border:'none',borderRadius:6}}>{loading ? 'Traitement...' : 'Passer commande (simulation)'}</button>
        </div>
      </div>
    </div>
  )
}
 
pages/api/suppliers.js
export default function handler(req, res) {
  const products = [
    {
      id: 'p-watch-01',
      title: 'Montre minimaliste',
      price: 49.9,
      img: '/sample-images/watch.jpg',
      suppliers: [
        { id: 'bigbuy-001', name: 'BigBuy (ES)', country: 'ES', leadDays: 1, price: 28, stock: 120, link: 'https://www.bigbuy.eu' },
        { id: 'cj-eu-01', name: 'CJdropshipping (FR warehouse)', country: 'FR', leadDays: 2, price: 25, stock: 300, link: 'https://fr.cjdropshipping.com' },
        { id: 'cn-std', name: 'CN Supplier', country: 'CN', leadDays: 8, price: 12, stock: 1000, link: 'https://www.aliexpress.com' },
      ]
    },
    {
      id: 'p-backpack-01',
      title: 'Sac à dos urbain',
      price: 69.9,
      img: '/sample-images/backpack.jpg',
      suppliers: [
        { id: 'bb-002', name: 'BigBuy (ES)', country: 'ES', leadDays: 1, price: 40, stock: 60, link: 'https://www.bigbuy.eu' },
        { id: 'spocket-eu-01', name: 'Spocket EU (DE)', country: 'DE', leadDays: 3, price: 42, stock: 30, link: 'https://www.spocket.co' },
        { id: 'cn-std2', name: 'CN Supplier', country: 'CN', leadDays: 10, price: 22, stock: 800, link: 'https://www.aliexpress.com' },
      ]
    },
    {
      id: 'p-lamp-01',
      title: 'Lampe LED tactile',
      price: 34.9,
      img: '/sample-images/lamp.jpg',
      suppliers: [
        { id: 'printful-eu', name: 'Printful EU', country: 'LV', leadDays: 2, price: 20, stock: 50, link: 'https://www.printful.com' },
        { id: 'cj-cn', name: 'CJ CN', country: 'CN', leadDays: 8, price: 10, stock: 1000, link: 'https://www.cjdropshipping.com' },
      ]
    }
  ]
  res.status(200).json({ products })
}
 
lib/shipping.js
export function estimateDeliveryDays(supplier, destCountry = 'FR') {
  if (supplier.country === destCountry) return supplier.leadDays + 1
  const eu = ['FR','DE','ES','IT','NL','BE','LU','PT','LV']
  if (eu.includes(supplier.country) && eu.includes(destCountry)) return supplier.leadDays + 2
  return supplier.leadDays + 7
}

export function pickBestSupplier(productSuppliers, destCountry='FR', preferFast=true) {
  const available = productSuppliers.filter(s => s.stock > 0)
  if (available.length === 0) return null
  const scored = available.map(s => ({ s, eta: estimateDeliveryDays(s, destCountry), cost: s.price }))
  if (preferFast) scored.sort((a,b) => a.eta - b.eta || a.cost - b.cost)
  else scored.sort((a,b) => a.cost - b.cost || a.eta - b.eta)
  return scored[0]
}
 
lib/cj.js
import axios from 'axios'
const BASE = process.env.CJ_BASE_URL || 'https://developers.cjdropshipping.com/api'
const CLIENT_ID = process.env.CJ_CLIENT_ID
const CLIENT_SECRET = process.env.CJ_CLIENT_SECRET
let tokenCache = { token: null, expiresAt: 0 }

async function getToken() {
  const now = Date.now()
  if (tokenCache.token && tokenCache.expiresAt > now + 60000) return tokenCache.token
  const resp = await axios.post(`${BASE}/oauth/token`, { client_id: CLIENT_ID, client_secret: CLIENT_SECRET, grant_type: 'client_credentials' })
  tokenCache.token = resp.data.access_token
  tokenCache.expiresAt = now + (resp.data.expires_in || 3600) * 1000
  return tokenCache.token
}

export async function cjCreateOrder(payload) {
  const token = await getToken()
  const r = await axios.post(`${BASE}/orders/createOrder`, payload, { headers: { Authorization: `Bearer ${token}` } })
  return r.data
}

export async function cjGetProduct(sku) {
  const token = await getToken()
  const r = await axios.get(`${BASE}/products/${sku}`, { headers: { Authorization: `Bearer ${token}` } })
  return r.data
}
 
lib/stripe.js
import Stripe from 'stripe'
const stripe = new Stripe(process.env.STRIPE_SECRET_KEY)

export async function createStripeSession({ line_items, success_url, cancel_url, metadata }) {
  const session = await stripe.checkout.sessions.create({ payment_method_types: ['card'], line_items, mode: 'payment', success_url, cancel_url, metadata, locale: 'fr' })
  return session
}

export async function verifyStripeWebhookSignature(req) {
  const sig = req.headers['stripe-signature']
  const buf = await buffer(req)
  try {
    const event = stripe.webhooks.constructEvent(buf, sig, process.env.STRIPE_WEBHOOK_SECRET)
    return { ok: true, event }
  } catch (err) {
    return { ok: false, error: err }
  }
}

async function buffer(req) {
  return new Promise((resolve, reject) => {
    const chunks = []
    req.on('data', (c) => chunks.push(c))
    req.on('end', () => resolve(Buffer.concat(chunks)))
    req.on('error', reject)
  })
}
 
lib/easypost.js
import axios from 'axios'
const BASE = 'https://api.easypost.com/v2'
const KEY = process.env.EASYPOST_API_KEY

export async function createShipmentAndBuyLabel({ to_address, from_address, parcel }) {
  const auth = { username: KEY, password: '' }
  const shipResp = await axios.post(`${BASE}/shipments`, { shipment: { to_address, from_address, parcel } }, { auth })
  const shipment = shipResp.data
  const rates = shipment.rates || []
  if (rates.length === 0) return shipment
  const chosen = rates.sort((a,b) => parseFloat(a.rate) - parseFloat(b.rate))[0]
  const buyResp = await axios.post(`${BASE}/shipments/${shipment.id}/buy`, { rate: chosen }, { auth })
  return buyResp.data
}
 
pages/api/order.js
import axios from 'axios'
import { createStripeSession } from '../../lib/stripe'
import { pickBestSupplier } from '../../lib/shipping'

export default async function handler(req, res) {
  if (req.method !== 'POST') return res.status(405).end()
  const { cart, customer } = req.body
  if (!cart || cart.length === 0) return res.status(400).json({ error: 'cart empty' })

  const { data } = await axios.get(`${process.env.NEXT_PUBLIC_BASE_URL || ''}/api/suppliers`)
  const products = data.products

  const assignments = cart.map(item => {
    const prod = products.find(p => p.id === item.id)
    const best = pickBestSupplier(prod.suppliers, customer.country || 'FR', true)
    return { item, product: prod, chosen: best }
  })

  const internalOrder = { id: 'INT-' + Date.now(), customer, assignments, status: 'pending_payment' }

  const line_items = cart.map(it => ({ price_data: { currency: 'eur', product_data: { name: it.title }, unit_amount: Math.round(it.price * 100) }, quantity: it.qty }))
  const session = await createStripeSession({ line_items, success_url: `${process.env.NEXT_PUBLIC_BASE_URL}/success?session_id={CHECKOUT_SESSION_ID}`, cancel_url: `${process.env.NEXT_PUBLIC_BASE_URL}/cart`, metadata: { internalOrderId: internalOrder.id } })

  // TODO: save internalOrder + session.id in DB

  return res.status(200).json({ checkoutUrl: session.url, internalOrder })
}
 
pages/api/webhooks/stripe.js
import { verifyStripeWebhookSignature } from '../../../lib/stripe'

export const config = { api: { bodyParser: false } }

export default async function handler(req, res) {
  const { ok, event, error } = await verifyStripeWebhookSignature(req)
  if (!ok) return res.status(400).send(`Webhook Error: ${error.message}`)

  if (event.type === 'checkout.session.completed') {
    const session = event.data.object
    const internalOrderId = session.metadata?.internalOrderId
    console.log('Payment received for internalOrder', internalOrderId)
    // TODO: 1) Mark order paid in DB
    // 2) For each assignment, call supplier API (CJ / BigBuy / Printful) to create supplier order
    // 3) If supplier provides tracking, store and email customer
  }

  res.status(200).end()
}
 
public/sample-images
Place any three images named watch.jpg, backpack.jpg, lamp.jpg in /public/sample-images/ or modifiez les URLs dans pages/api/suppliers.js pour pointer vers des images externes.
 
Instructions pour créer le repo GitHub & déployer
1.	git init → git add . → git commit -m "Initial commit"
2.	Crée un repo sur GitHub et pousse :
git remote add origin https://github.com/TonUser/dropship-fast.git
git branch -M main
git push -u origin main
3.	Sur Vercel : New Project → Import GitHub repo → Configure env vars (colle le contenu de .env.example dans Vercel) → Deploy.
4.	Pour Stripe Webhooks : configure l’URL https://<ton-domaine>/api/webhooks/stripe et coller STRIPE_WEBHOOK_SECRET dans Vercel.
