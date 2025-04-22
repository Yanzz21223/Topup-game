# Topup-game<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Top-Up Game</title>
  <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-gray-100">
  <div class="max-w-xl mx-auto p-6 bg-white mt-10 rounded-2xl shadow-md">
    <h1 class="text-2xl font-bold text-center mb-4">Top-Up Game</h1>

    <form id="topupForm" class="space-y-4">
      <div>
        <label class="block font-semibold">Username Admin</label>
        <input id="adminUsername" type="text" class="w-full p-2 border rounded" placeholder="Admin (jika login)" />
      </div>

      <div>
        <label class="block font-semibold">Pilih Game</label>
        <select id="game" class="w-full p-2 border rounded">
          <option value="MLBB">Mobile Legends</option>
          <option value="FF">Free Fire</option>
          <option value="PUBG">PUBG Mobile</option>
        </select>
      </div>

      <div>
        <label class="block font-semibold">ID Pemain</label>
        <input id="playerId" type="text" class="w-full p-2 border rounded" placeholder="Masukkan ID" required />
      </div>

      <div>
        <label class="block font-semibold">Nominal Top-Up</label>
        <select id="nominal" class="w-full p-2 border rounded">
          <option value="10" data-price="3000">10 Diamonds - Rp3.000</option>
          <option value="50" data-price="15000">50 Diamonds - Rp15.000</option>
          <option value="100" data-price="30000">100 Diamonds - Rp30.000</option>
        </select>
      </div>

      <div>
        <label class="block font-semibold">Metode Pembayaran</label>
        <select id="paymentMethod" class="w-full p-2 border rounded">
          <option value="Dana">Dana</option>
          <option value="OVO">OVO</option>
          <option value="GoPay">GoPay</option>
          <option value="Transfer Bank">Transfer Bank</option>
        </select>
      </div>

      <div>
        <label class="block font-semibold">Kode Promo (Opsional)</label>
        <input id="promoCode" type="text" class="w-full p-2 border rounded" placeholder="Contoh: DISKON10" />
      </div>

      <div>
        <label class="block font-semibold">Catatan (Opsional)</label>
        <textarea id="note" class="w-full p-2 border rounded" placeholder="Contoh: Kirim cepat ya..."></textarea>
      </div>

      <div>
        <p class="font-semibold">Total Harga: <span id="totalPrice">Rp0</span></p>
      </div>

      <button type="submit" class="w-full bg-blue-600 text-white py-2 rounded hover:bg-blue-700">Bayar</button>
    </form>

    <div id="output" class="mt-6 text-center font-medium"></div>

    <div class="mt-10">
      <h2 class="text-xl font-bold mb-2">Riwayat Transaksi</h2>
      <ul id="transactionHistory" class="space-y-2 text-sm text-gray-700"></ul>
    </div>

    <div class="mt-10">
      <h2 class="text-xl font-bold mb-2">Promo Aktif</h2>
      <ul class="text-sm list-disc pl-5 text-gray-700" id="promoList"></ul>
    </div>
  </div>

  <script>
    const nominalSelect = document.getElementById('nominal');
    const totalPriceSpan = document.getElementById('totalPrice');
    const transactionHistory = document.getElementById('transactionHistory');
    const promoInput = document.getElementById('promoCode');
    const promoList = document.getElementById('promoList');

    const promoCodes = {
      'DISKON10': { discount: 0.1, expires: '2025-12-31' },
      'DISKON20': { discount: 0.2, expires: '2025-06-30' },
      'GRATISONGKIR': { discount: 0.05, expires: '2025-05-31' }
    };

    function isPromoValid(promoCode) {
      const promo = promoCodes[promoCode];
      if (!promo) return false;
      const now = new Date();
      const expiry = new Date(promo.expires);
      return now <= expiry;
    }

    function getPromoDiscount(code) {
      const upperCode = code.toUpperCase();
      if (promoCodes[upperCode] && isPromoValid(upperCode)) {
        return promoCodes[upperCode].discount;
      }
      return 0;
    }

    function updatePrice() {
      const selectedOption = nominalSelect.options[nominalSelect.selectedIndex];
      const basePrice = parseInt(selectedOption.getAttribute('data-price'));
      const promoCode = promoInput.value.toUpperCase();
      const discount = getPromoDiscount(promoCode);
      const finalPrice = basePrice * (1 - discount);
      totalPriceSpan.textContent = `Rp${Math.round(finalPrice).toLocaleString('id-ID')}`;
    }

    function loadTransactions() {
      const stored = JSON.parse(localStorage.getItem('transactions') || '[]');
      stored.reverse().forEach(addTransaction);
    }

    function saveTransaction(data) {
      const stored = JSON.parse(localStorage.getItem('transactions') || '[]');
      stored.push(data);
      localStorage.setItem('transactions', JSON.stringify(stored));
    }

    function addTransaction(data) {
      const li = document.createElement('li');
      li.textContent = `${data.date} - ${data.game}, ID: ${data.playerId}, ${data.nominal} Diamonds, Rp${data.price}, ${data.paymentMethod}${data.note ? ', Catatan: ' + data.note : ''}`;
      transactionHistory.prepend(li);
    }

    function showActivePromos() {
      promoList.innerHTML = '';
      Object.entries(promoCodes).forEach(([code, data]) => {
        if (isPromoValid(code)) {
          const li = document.createElement('li');
          li.textContent = `${code} - Diskon ${data.discount * 100}% sampai ${data.expires}`;
          promoList.appendChild(li);
        }
      });
    }

    nominalSelect.addEventListener('change', updatePrice);
    promoInput.addEventListener('input', updatePrice);
    window.addEventListener('DOMContentLoaded', () => {
      updatePrice();
      loadTransactions();
      showActivePromos();
    });

    document.getElementById('topupForm').addEventListener('submit', function(e) {
      e.preventDefault();

      const game = document.getElementById('game').value;
      const playerId = document.getElementById('playerId').value;
      const nominal = document.getElementById('nominal').value;
      const basePrice = parseInt(document.getElementById('nominal').selectedOptions[0].getAttribute('data-price'));
      const paymentMethod = document.getElementById('paymentMethod').value;
      const promoCode = promoInput.value.toUpperCase();
      const discount = getPromoDiscount(promoCode);
      const note = document.getElementById('note').value;
      const finalPrice = Math.round(basePrice * (1 - discount));
      const date = new Date().toLocaleString('id-ID');

      const confirmTopup = confirm(
        `Konfirmasi Top-Up:\nGame: ${game}\nID: ${playerId}\nNominal: ${nominal} Diamonds\nHarga: Rp${finalPrice.toLocaleString('id-ID')}\nDiskon: ${discount * 100}%\nPembayaran: ${paymentMethod}${note ? '\nCatatan: ' + note : ''}\n\nLanjutkan?`
      );

      if (confirmTopup) {
        document.getElementById('output').innerText = 
          `Top-up berhasil!\nGame: ${game}\nID: ${playerId}\nJumlah: ${nominal} Diamonds\nMetode: ${paymentMethod}\nTotal: Rp${finalPrice.toLocaleString('id-ID')}${note ? '\nCatatan: ' + note : ''}`;

        const transactionData = { date, game, playerId, nominal, price: finalPrice.toLocaleString('id-ID'), paymentMethod, note };
        addTransaction(transactionData);
        saveTransaction(transactionData);
      } else {
        document.getElementById('output').innerText = 'Top-up dibatalkan.';
      }
    });
  </script>
</body>
</html>
