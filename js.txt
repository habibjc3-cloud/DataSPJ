const WEB_APP_URL =
  "https://script.google.com/macros/s/AKfycbzc37xjEXFoWqJ4nM5hiUt5Rv4KJSFI__WPvHC00NPWlFusnyd3ipD-hkwy7cP3K1HV/exec";

let selectedProjectValue = "";

function toggleProjectDropdown() {
  const menu = document.getElementById("dropdownMenu");
  menu.classList.toggle("show");
}

function selectProject(label, value) {
  document.getElementById("selectedOption").innerText = label;
  document.getElementById("dropdownMenu").classList.remove("show");
  selectedProjectValue = label;

  const btnLanjut = document.getElementById("btnLanjut");
  btnLanjut.classList.add("visible");
}

function toggleNamaDropdown() {
  const menu = document.getElementById("dropdownNamaMenu");
  menu.classList.toggle("show");
}

function selectNama(nama) {
  document.getElementById("selectedNama").innerText = nama;
  document.getElementById("inputNamaValue").value = nama;
  document.getElementById("dropdownNamaMenu").classList.remove("show");
  checkFormStatus();
}

function toggleJenisDropdown() {
  const menu = document.getElementById("dropdownJenisMenu");
  menu.classList.toggle("show");
}

function selectJenis(jenis) {
  document.getElementById("selectedJenis").innerText = jenis;
  document.getElementById("inputJenisValue").value = jenis;
  document.getElementById("dropdownJenisMenu").classList.remove("show");
  checkFormStatus();
}

function hitungTotal() {
  const jumlah = parseFloat(document.getElementById("inputJumlah").value) || 0;
  const hargaSatuan =
    parseFloat(document.getElementById("inputHargaSatuan").value) || 0;
  const total = jumlah * hargaSatuan;

  const totalInput = document.getElementById("inputTotalHarga");
  if (total > 0) {
    totalInput.value = "Rp " + total.toLocaleString("id-ID");
  } else {
    totalInput.value = "";
  }
}

function handleFileUpload(input) {
  const label = document.getElementById("uploadLabel");
  const text = document.getElementById("uploadText");

  if (input.files && input.files[0]) {
    const fileName = input.files[0].name;
    label.classList.remove("btn-red");
    label.classList.add("btn-green");
    text.innerText = "✓ TERUNGGAH (" + fileName.substring(0, 10) + "...)";
  } else {
    label.classList.remove("btn-green");
    label.classList.add("btn-red");
    text.innerText = "UNGGAH BUKTI";
  }
  checkFormStatus();
}

function checkFormStatus() {
  const nama = document.getElementById("inputNamaValue").value;
  const tanggal = document.getElementById("inputTanggal")
    ? document.getElementById("inputTanggal").value
    : "";
  const deskripsi = document.getElementById("inputDeskripsi").value.trim();
  const jenis = document.getElementById("inputJenisValue").value;
  const jumlah = document.getElementById("inputJumlah").value;
  const satuan = document.getElementById("inputSatuan").value.trim();
  const hargaSatuan = document.getElementById("inputHargaSatuan").value;
  const file = document.getElementById("fileBukti").files.length;

  const btnSelesai = document.getElementById("btnSelesai");

  if (
    nama &&
    tanggal &&
    deskripsi &&
    jenis &&
    jumlah &&
    satuan &&
    hargaSatuan &&
    file > 0
  ) {
    btnSelesai.classList.remove("btn-red");
    btnSelesai.classList.add("btn-green");
    btnSelesai.disabled = false;
  } else {
    btnSelesai.classList.remove("btn-green");
    btnSelesai.classList.add("btn-red");
    btnSelesai.disabled = true;
  }
}

// FUNGSI UNTUK MENGAMBIL DATA RANKING DARI SPREADSHEET
function loadRanking() {
  const container = document.querySelector(".ranking-content");
  if (!container) return;

  container.innerHTML = "<p class='ranking-placeholder'>Memuat ranking...</p>";

  fetch(WEB_APP_URL)
    .then((res) => res.json())
    .then((data) => {
      if (
        data.result === "success" &&
        data.ranking &&
        data.ranking.length > 0
      ) {
        let html = "";
        data.ranking.forEach((item, index) => {
          // Jumlah bintang berdasarkan ranking (Juara 1 = ⭐⭐⭐, Juara 2 = ⭐⭐, dst)
          let stars = "";
          if (index === 0) stars = " ⭐⭐⭐";
          else if (index === 1) stars = " ⭐⭐";
          else if (index === 2) stars = " ⭐";

          const formattedTotal = "Rp " + item.total.toLocaleString("id-ID");

          html += `
            <div class="ranking-item">
              <span class="ranking-name">${item.nama.toUpperCase()} : ${formattedTotal}</span>
              <span class="ranking-stars">${stars}</span>
            </div>
          `;
        });
        container.innerHTML = html;
      } else {
        container.innerHTML =
          "<p class='ranking-placeholder'>Belum ada data transaksi.</p>";
      }
    })
    .catch((err) => {
      console.error(err);
      container.innerHTML =
        "<p class='ranking-placeholder'>Gagal memuat ranking.</p>";
    });
}

function openModal() {
  document.getElementById("modalOverlay").classList.add("active");
  const tglInput = document.getElementById("inputTanggal");
  if (tglInput && !tglInput.value) {
    const today = new Date().toISOString().split("T")[0];
    tglInput.value = today;
  }
  checkFormStatus();
  loadRanking(); // Panggil data ranking setiap kali modal dibuka
}

function closeModal() {
  document.getElementById("modalOverlay").classList.remove("active");
}

function handleFormSubmit(event) {
  event.preventDefault();

  const btnSelesai = document.getElementById("btnSelesai");
  btnSelesai.innerText = "MENGIRIM DATA...";
  btnSelesai.disabled = true;

  const fileInput = document.getElementById("fileBukti").files[0];
  const reader = new FileReader();

  reader.onload = function (e) {
    const base64File = e.target.result;

    const payload = {
      project: selectedProjectValue,
      pic: document.getElementById("inputNamaValue").value,
      tanggal: document.getElementById("inputTanggal").value,
      deskripsi: document.getElementById("inputDeskripsi").value,
      jenis: document.getElementById("inputJenisValue").value,
      jumlah: document.getElementById("inputJumlah").value,
      satuan: document.getElementById("inputSatuan").value,
      hargaSatuan: document.getElementById("inputHargaSatuan").value,
      fileName: fileInput.name,
      fileData: base64File,
    };

    fetch(WEB_APP_URL, {
      method: "POST",
      body: JSON.stringify(payload),
    })
      .then((res) => res.json())
      .then((data) => {
        if (data.result === "success") {
          alert("Data dan Nota Berhasil Terupload!");
          resetForm();
          loadRanking(); // Reload ranking setelah data baru dikirim
          closeModal();
        } else {
          alert("Gagal menyimpan data: " + data.error);
        }
      })
      .catch((err) => {
        console.error(err);
        alert("Terjadi kesalahan koneksi!");
      })
      .finally(() => {
        btnSelesai.innerText = "SELESAI";
        btnSelesai.disabled = false;
      });
  };

  reader.readAsDataURL(fileInput);
}

function resetForm() {
  document.getElementById("spjForm").reset();
  document.getElementById("inputNamaValue").value = "";
  document.getElementById("inputJenisValue").value = "";
  document.getElementById("selectedNama").innerText = "NAMA";
  document.getElementById("selectedJenis").innerText =
    "PENGELUARAN / PEMASUKAN";
  document.getElementById("uploadLabel").classList.remove("btn-green");
  document.getElementById("uploadLabel").classList.add("btn-red");
  document.getElementById("uploadText").innerText = "UNGGAH BUKTI";
  checkFormStatus();
}

window.onclick = function (event) {
  if (
    !event.target.matches(".dropdown-button") &&
    !event.target.matches("#selectedOption")
  ) {
    const menuProject = document.getElementById("dropdownMenu");
    if (menuProject && menuProject.classList.contains("show")) {
      menuProject.classList.remove("show");
    }
  }

  if (
    !event.target.matches(".dropdown-button-modal") &&
    !event.target.matches("#selectedNama")
  ) {
    const menuNama = document.getElementById("dropdownNamaMenu");
    if (menuNama && menuNama.classList.contains("show")) {
      menuNama.classList.remove("show");
    }
  }

  if (
    !event.target.matches(".dropdown-button-modal") &&
    !event.target.matches("#selectedJenis")
  ) {
    const menuJenis = document.getElementById("dropdownJenisMenu");
    if (menuJenis && menuJenis.classList.contains("show")) {
      menuJenis.classList.remove("show");
    }
  }
};
