# https-your-app-name.vercel.app

import { useState, useEffect } from "react"; import { Card, CardContent } from "@/components/ui/card"; import { Button } from "@/components/ui/button"; import { initializeApp } from "firebase/app"; import { getFirestore, collection, addDoc, onSnapshot, updateDoc, doc } from "firebase/firestore"; import { getAuth, RecaptchaVerifier, signInWithPhoneNumber, signOut } from "firebase/auth"; import { Bar } from "react-chartjs-2";
const firebaseConfig = { apiKey: "YOUR_KEY", authDomain: "YOUR_DOMAIN", projectId: "YOUR_PROJECT_ID", };
const app = initializeApp(firebaseConfig); const db = getFirestore(app); const auth = getAuth(app);
export default function MeatBookingApp() { const [orders, setOrders] = useState([]); const [location, setLocation] = useState(null); const [deliveryPersons, setDeliveryPersons] = useState([ { name: "Ravi", phone: "9999999991" }, { name: "Akram", phone: "9999999992" }, ]); const [form, setForm] = useState({ name: "", mobile: "", address: "", city: "Hyderabad", meatQty: "", chickenQty: "", beefQty: "" });
const prices = { meat: 700, chicken: 250, beef: 350 };
useEffect(() => { const unsub = onSnapshot(collection(db, "orders"), (snap) => { setOrders(snap.docs.map(doc => ({ id: doc.id, ...doc.data() }))); });
navigator.geolocation?.getCurrentPosition((pos) => {
  setLocation({ lat: pos.coords.latitude, lng: pos.coords.longitude });
});


return () => unsub();
}, []);
const calculateTotal = () => (form.meatQty || 0) * prices.meat + (form.chickenQty || 0) * prices.chicken + (form.beefQty || 0) * prices.beef;
// 🔥 AI DEMAND PREDICTION (simple) const predictDemand = () => { const totalOrders = orders.length; if (totalOrders < 5) return "Low Demand"; if (totalOrders < 15) return "Medium Demand"; return "High Demand"; };
// 🚚 Assign delivery person const assignDelivery = () => { return deliveryPersons[Math.floor(Math.random() * deliveryPersons.length)]; };
const handleSubmit = async () => { const total = calculateTotal(); const delivery = assignDelivery();
await addDoc(collection(db, "orders"), {
  ...form,
  total,
  status: "pending",
  deliveryPerson: delivery.name,
  deliveryPhone: delivery.phone,
  tracking: location,
});
};
const updateStatus = async (id, status) => { await updateDoc(doc(db, "orders", id), { status }); };
const data = { labels: ["Pending", "Completed"], datasets: [{ label: "Orders", data: [orders.filter(o=>o.status==='pending').length, orders.filter(o=>o.status==='completed').length], backgroundColor:["orange","green"] }] };
return ( 
  {/* CUSTOMER ORDER */}
  <Card><CardContent>
    <h2>Place Order</h2>
    <input placeholder="Name" onChange={e=>setForm({...form,name:e.target.value})} />    <input placeholder="Mobile" onChange={e=>setForm({...form,mobile:e.target.value})} />
    <input placeholder="Address" onChange={e=>setForm({...form,address:e.target.value})} />
    <input placeholder="Meat kg" onChange={e=>setForm({...form,meatQty:e.target.value})} />
    <input placeholder="Chicken kg" onChange={e=>setForm({...form,chickenQty:e.target.value})} />
    <input placeholder="Beef kg" onChange={e=>setForm({...form,beefQty:e.target.value})} />
    <p>Total ₹{calculateTotal()}</p>
    <Button onClick={handleSubmit}>Order</Button>
  </CardContent></Card>
  {/* GOOGLE MAP VIEW (BASIC) */}  <div>
    <h3>Live Tracking (Google Maps)</h3>    {location && (
      <iframe
        width="100%"
        height="300"
        src={`https://maps.google.com/maps?q=${location.lat},${location.lng}&z=15&output=embed`}      />    )}
  </div>
  {/* ADMIN DASHBOARD */}
  <div>    <h2>Dashboard</h2>    <Bar data={data} />
    <p>AI Demand: {predictDemand()}</p>
    {orders.map(o => (      <div key={o.id} className="border p-2">
        {o.name} | {o.status} | Delivery: {o.deliveryPerson}        <Button onClick={()=>updateStatus(o.id,"completed")}>Complete</Button>
      </div>
    ))}
  </div>
  {/* DELIVERY TRACKING */}
  <div>
    <h3>Delivery Tracking</h3>
    {orders.map(o => (
      <div key={o.id}>
        {o.name} → {o.deliveryPerson} ({o.deliveryPhone})
      </div>
    ))}
  </div>
</div>
); }
