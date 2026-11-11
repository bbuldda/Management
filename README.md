<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>남지국민체육센터 - 스마트 자산 관리 시스템</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Html5-Qrcode CDN -->
    <script src="https://unpkg.com/html5-qrcode" type="text/javascript"></script>
    <!-- Lucide Icons CDN -->
    <script src="https://unpkg.com/lucide@latest"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Pretendard:wght@300;400;500;600;700&display=swap');
        body { font-family: 'Pretendard', sans-serif; }
    </style>
</head>
<body class="bg-slate-50 text-slate-800 min-h-screen flex flex-col">

    <!-- 상단 헤더 -->
    <header class="bg-blue-600 text-white shadow-md sticky top-0 z-50">
        <div class="max-w-7xl mx-auto px-4 py-3 flex justify-between items-center">
            <div class="flex items-center space-x-2">
                <i data-lucide="package-search" class="w-7 h-7"></i>
                <h1 class="text-lg font-bold">남지국민체육센터 자산관리</h1>
            </div>
            <div class="flex space-x-2">
                <button onclick="switchTab('scanner')" id="btn-tab-scanner" class="px-3 py-1.5 rounded-lg text-sm font-medium bg-blue-700 transition flex items-center space-x-1">
                    <i data-lucide="scan" class="w-4 h-4"></i><span>바코드/조회</span>
                </button>
                <button onclick="switchTab('list')" id="btn-tab-list" class="px-3 py-1.5 rounded-lg text-sm font-medium hover:bg-blue-700 transition flex items-center space-x-1">
                    <i data-lucide="list" class="w-4 h-4"></i><span>자산목록</span>
                </button>
            </div>
        </div>
    </header>

    <!-- 메인 컨테이너 -->
    <main class="flex-1 max-w-7xl w-full mx-auto p-4 md:p-6">

        <!-- [TAB 1] 바코드 스캔 및 조회 화면 -->
        <section id="tab-scanner" class="space-y-6">
            <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-100 grid grid-cols-1 md:grid-cols-2 gap-6">
                <!-- 카메라 스캐너 영역 -->
                <div class="flex flex-col items-center justify-center bg-slate-900 rounded-xl p-4 text-white relative overflow-hidden min-h-[300px]">
                    <div id="reader" class="w-full rounded-lg overflow-hidden"></div>
                    <div id="scanner-placeholder" class="text-center py-12">
                        <i data-lucide="camera-off" class="w-12 h-12 mx-auto text-slate-500 mb-2"></i>
                        <p class="text-sm text-slate-400">카메라 스캐너가 대기 중입니다.</p>
                        <button onclick="startScanner()" class="mt-4 px-4 py-2 bg-blue-600 hover:bg-blue-500 rounded-lg text-sm font-medium transition">카메라 스캔 시작</button>
                    </div>
                    <button id="stop-scanner-btn" onclick="stopScanner()" class="hidden mt-3 px-3 py-1.5 bg-red-600 hover:bg-red-500 rounded-lg text-xs font-medium transition">스캔 중지</button>
                </div>

                <!-- 수동 입력 및 결과 안내 -->
                <div class="flex flex-col justify-between space-y-4">
                    <div>
                        <h2 class="text-lg font-bold text-slate-900 mb-2">바코드 직접 입력 또는 조회</h2>
                        <p class="text-xs text-slate-500 mb-4">스티커의 바코드 번호를 직접 입력하거나 스캔하여 자산 실물 및 정보를 확인하세요.</p>
                        <div class="flex space-x-2">
                            <input type="text" id="manual-barcode" placeholder="예: 120802800100005" class="flex-1 px-4 py-2 border border-slate-200 rounded-xl focus:outline-none focus:ring-2 focus:ring-blue-500 text-sm">
                            <button onclick="searchBarcode(document.getElementById('manual-barcode').value)" class="px-5 py-2 bg-slate-800 hover:bg-slate-700 text-white rounded-xl text-sm font-medium transition">조회</button>
                        </div>
                    </div>

                    <!-- 파일 업로드로 바코드 읽기 -->
                    <div class="border-t pt-4">
                        <label class="block text-xs font-medium text-slate-600 mb-2">바코드 이미지 파일 업로드 인식</label>
                        <input type="file" id="barcode-file-input" accept="image/*" onchange="scanFileBarcode(event)" class="w-full text-xs text-slate-500 file:mr-4 file:py-2 file:px-4 file:rounded-xl file:border-0 file:text-xs file:font-semibold file:bg-blue-50 file:text-blue-700 hover:file:bg-blue-100 cursor-pointer">
                    </div>
                </div>
            </div>

            <!-- 스캔/조회 결과 출력 영역 -->
            <div id="search-result-container" class="hidden bg-white p-6 rounded-2xl shadow-sm border border-slate-100 animate-fade-in">
                <div class="flex justify-between items-start pb-4 border-b">
                    <div>
                        <span id="result-badge" class="px-2.5 py-1 rounded-full text-xs font-semibold">등록된 자산</span>
                        <h3 id="res-asset-name" class="text-xl font-bold text-slate-900 mt-1">-</h3>
                        <p id="res-barcode" class="text-xs text-slate-400 font-mono mt-0.5">-</p>
                    </div>
                    <button onclick="openEditModalFromScan()" class="px-3 py-1.5 bg-blue-50 text-blue-600 hover:bg-blue-100 rounded-lg text-xs font-medium transition flex items-center space-x-1">
                        <i data-lucide="edit-3" class="w-3.5 h-3.5"></i><span>정보 및 사진 수정</span>
                    </button>
                </div>

                <div class="grid grid-cols-1 md:grid-cols-3 gap-6 mt-6">
                    <!-- 자산 실물 사진 -->
                    <div class="flex flex-col items-center justify-center bg-slate-100 rounded-xl p-2 border border-slate-200 min-h-[220px] relative">
                        <img id="res-photo" src="" alt="실물 사진 없음" class="max-h-64 object-contain rounded-lg hidden">
                        <div id="res-no-photo" class="text-center p-6 text-slate-400">
                            <i data-lucide="image-off" class="w-10 h-10 mx-auto mb-2 text-slate-400"></i>
                            <p class="text-xs">등록된 실물 사진이 없습니다.<br>사진을 등록해 현장에서 물건을 쉽게 찾으세요.</p>
                        </div>
                    </div>

                    <!-- 상세 정보 -->
                    <div class="md:col-span-2 grid grid-cols-2 gap-4 text-sm content-center">
                        <div class="bg-slate-50 p-3 rounded-xl border border-slate-100">
                            <span class="block text-xs text-slate-400">자산번호</span>
                            <span id="res-asset-no" class="font-semibold text-slate-700">-</span>
                        </div>
                        <div class="bg-slate-50 p-3 rounded-xl border border-slate-100">
                            <span class="block text-xs text-slate-400">회계명</span>
                            <span id="res-account-name" class="font-semibold text-slate-700">-</span>
                        </div>
                        <div class="bg-slate-50 p-3 rounded-xl border border-slate-100">
                            <span class="block text-xs text-slate-400">규격</span>
                            <span id="res-spec" class="font-semibold text-slate-700">-</span>
                        </div>
                        <div class="bg-slate-50 p-3 rounded-xl border border-slate-100">
                            <span class="block text-xs text-slate-400">취득일자</span>
                            <span id="res-date" class="font-semibold text-slate-700">-</span>
                        </div>
                        <div class="col-span-2 bg-slate-50 p-3 rounded-xl border border-slate-100">
                            <span class="block text-xs text-slate-400">취득금액</span>
                            <span id="res-price" class="font-bold text-blue-600 text-base">-</span>
                        </div>
                    </div>
                </div>
            </div>
        </section>

        <!-- [TAB 2] 자산 목록 관리 화면 -->
        <section id="tab-list" class="hidden space-y-4">
            <div class="bg-white p-4 rounded-2xl shadow-sm border border-slate-100 flex flex-col md:flex-row justify-between items-center gap-3">
                <div class="relative w-full md:w-80">
                    <i data-lucide="search" class="absolute left-3 top-2.5 w-4 h-4 text-slate-400"></i>
                    <input type="text" id="search-keyword" oninput="renderAssetTable()" placeholder="자산명, 바코드, 번호 검색..." class="w-full pl-9 pr-4 py-2 border border-slate-200 rounded-xl text-sm focus:outline-none focus:ring-2 focus:ring-blue-500">
                </div>
                <div class="flex flex-wrap gap-2 w-full md:w-auto justify-end">
                    <button onclick="openAddModal()" class="px-4 py-2 bg-blue-600 hover:bg-blue-500 text-white rounded-xl text-sm font-medium transition flex items-center space-x-1">
                        <i data-lucide="plus" class="w-4 h-4"></i><span>신규 자산 등록</span>
                    </button>
                    <button onclick="exportCSV()" class="px-4 py-2 bg-slate-100 hover:bg-slate-200 text-slate-700 rounded-xl text-sm font-medium transition flex items-center space-x-1">
                        <i data-lucide="download" class="w-4 h-4"></i><span>CSV 내보내기</span>
                    </button>
                    <label class="px-4 py-2 bg-slate-100 hover:bg-slate-200 text-slate-700 rounded-xl text-sm font-medium transition flex items-center space-x-1 cursor-pointer">
                        <i data-lucide="upload" class="w-4 h-4"></i><span>CSV 가져오기</span>
                        <input type="file" accept=".csv" onchange="importCSV(event)" class="hidden">
                    </label>
                </div>
            </div>

            <!-- 자산 리스트 테이블/카드 -->
            <div class="bg-white rounded-2xl shadow-sm border border-slate-100 overflow-hidden">
                <div class="overflow-x-auto">
                    <table class="w-full text-left border-collapse">
                        <thead>
                            <tr class="bg-slate-50 border-b text-xs font-semibold text-slate-500 uppercase">
                                <th class="p-3">사진</th>
                                <th class="p-3">바코드 번호</th>
                                <th class="p-3">자산명</th>
                                <th class="p-3">자산번호</th>
                                <th class="p-3">규격</th>
                                <th class="p-3">취득일자</th>
                                <th class="p-3">취득금액</th>
                                <th class="p-3 text-center">관리</th>
                            </tr>
                        </thead>
                        <tbody id="asset-table-body" class="divide-y text-sm text-slate-700">
                            <!-- JS 렌더링 -->
                        </tbody>
                    </table>
                </div>
            </div>
        </section>

    </main>

    <!-- 자산 추가/수정 모달 -->
    <div id="asset-modal" class="fixed inset-0 bg-black/50 backdrop-blur-sm hidden items-center justify-center z-50 p-4 overflow-y-auto">
        <div class="bg-white rounded-2xl max-w-lg w-full p-6 shadow-xl space-y-4 my-8">
            <div class="flex justify-between items-center pb-3 border-b">
                <h3 id="modal-title" class="text-lg font-bold text-slate-900">신규 자산 등록</h3>
                <button onclick="closeModal()" class="text-slate-400 hover:text-slate-600"><i data-lucide="x" class="w-5 h-5"></i></button>
            </div>
            <form id="asset-form" onsubmit="saveAsset(event)" class="space-y-3 text-sm">
                <input type="hidden" id="edit-original-id">
                <div>
                    <label class="block text-xs font-semibold text-slate-600 mb-1">바코드 번호 *</label>
                    <input type="text" id="modal-barcode" required class="w-full px-3 py-2 border rounded-xl focus:ring-2 focus:ring-blue-500">
                </div>
                <div class="grid grid-cols-2 gap-3">
                    <div>
                        <label class="block text-xs font-semibold text-slate-600 mb-1">자산명 *</label>
                        <input type="text" id="modal-name" required class="w-full px-3 py-2 border rounded-xl focus:ring-2 focus:ring-blue-500">
                    </div>
                    <div>
                        <label class="block text-xs font-semibold text-slate-600 mb-1">자산번호</label>
                        <input type="text" id="modal-asset-no" class="w-full px-3 py-2 border rounded-xl focus:ring-2 focus:ring-blue-500">
                    </div>
                </div>
                <div class="grid grid-cols-2 gap-3">
                    <div>
                        <label class="block text-xs font-semibold text-slate-600 mb-1">규격</label>
                        <input type="text" id="modal-spec" class="w-full px-3 py-2 border rounded-xl focus:ring-2 focus:ring-blue-500">
                    </div>
                    <div>
                        <label class="block text-xs font-semibold text-slate-600 mb-1">회계명</label>
                        <input type="text" id="modal-account" value="남지국민체육센터" class="w-full px-3 py-2 border rounded-xl focus:ring-2 focus:ring-blue-500">
                    </div>
                </div>
                <div class="grid grid-cols-2 gap-3">
                    <div>
                        <label class="block text-xs font-semibold text-slate-600 mb-1">취득일자 (YYYY.MM.DD)</label>
                        <input type="text" id="modal-date" placeholder="2024.01.01" class="w-full px-3 py-2 border rounded-xl focus:ring-2 focus:ring-blue-500">
                    </div>
                    <div>
                        <label class="block text-xs font-semibold text-slate-600 mb-1">취득금액 (원)</label>
                        <input type="number" id="modal-price" class="w-full px-3 py-2 border rounded-xl focus:ring-2 focus:ring-blue-500">
                    </div>
                </div>

                <!-- 카메라 촬영 및 사진 첨부 -->
                <div>
                    <label class="block text-xs font-semibold text-slate-600 mb-1">자산 실물 사진 촬영/첨부</label>
                    <div class="flex items-center space-x-3">
                        <input type="file" id="modal-photo-file" accept="image/*" capture="environment" onchange="previewImage(event)" class="w-full text-xs text-slate-500 file:mr-2 file:py-1.5 file:px-3 file:rounded-lg file:border-0 file:text-xs file:font-semibold file:bg-blue-50 file:text-blue-700 hover:file:bg-blue-100 cursor-pointer">
                    </div>
                    <div class="mt-2 flex justify-center bg-slate-50 border rounded-xl p-2 h-36 relative overflow-hidden">
                        <img id="modal-photo-preview" src="" alt="미리보기" class="h-full object-contain hidden">
                        <span id="modal-no-photo-text" class="text-xs text-slate-400 self-center">사진이 없습니다.</span>
                    </div>
                    <input type="hidden" id="modal-photo-base64">
                </div>

                <div class="flex justify-end space-x-2 pt-3 border-t">
                    <button type="button" onclick="closeModal()" class="px-4 py-2 bg-slate-100 hover:bg-slate-200 text-slate-700 rounded-xl text-sm font-medium">취소</button>
                    <button type="submit" class="px-4 py-2 bg-blue-600 hover:bg-blue-500 text-white rounded-xl text-sm font-medium">저장하기</button>
                </div>
            </form>
        </div>
    </div>

    <!-- 스크립트 로직 -->
    <script>
        // 초기 35건 자산 데이터
        const INITIAL_ASSETS = [
            { id: "120802800100001", assetNo: "1-2-8-28-1-1", name: "금고", spec: "-", date: "2023.12.28", price: 583000, account: "남지국민체육센터", photo: "" },
            { id: "120802800100002", assetNo: "1-2-8-28-1-2", name: "사무용 노트북", spec: "-", date: "2023.12.29", price: 2563770, account: "남지국민체육센터", photo: "" },
            { id: "120802800100003", assetNo: "1-2-8-28-1-3", name: "탈의실 제습", spec: "-", date: "2023.12.29", price: 1628740, account: "남지국민체육센터", photo: "" },
            { id: "120802800100004", assetNo: "1-2-8-28-1-4", name: "CCTV용 TV", spec: "55인치", date: "2024.04.24", price: 883030, account: "남지국민체육센터", photo: "" },
            { id: "120802800100005", assetNo: "1-2-8-28-1-5", name: "냉장고", spec: "710*600*1640", date: "2024.05.03", price: 1044770, account: "남지국민체육센터", photo: "" },
            { id: "120802800100006", assetNo: "1-2-8-28-1-6", name: "모니터", spec: "TG 6240VP 23.8", date: "2024.05.03", price: 964400, account: "남지국민체육센터", photo: "" },
            { id: "120802800100007", assetNo: "1-2-8-28-1-7", name: "목욕의자", spec: "510*750", date: "2024.04.26", price: 94600, account: "남지국민체육센터", photo: "" },
            { id: "120802800100008", assetNo: "1-2-8-28-1-8", name: "민원용 선반", spec: "1800*280*1200", date: "2024.04.08", price: 495000, account: "남지국민체육센터", photo: "" },
            { id: "120802800100009", assetNo: "1-2-8-28-1-9", name: "민원용 선반", spec: "800*400*1900", date: "2024.04.08", price: 495000, account: "남지국민체육센터", photo: "" },
            { id: "120802800100010", assetNo: "1-2-8-28-1-10", name: "수영장 벤치", spec: "2000*400*450", date: "2024.05.03", price: 510000, account: "남지국민체육센터", photo: "" },
            { id: "120802800100011", assetNo: "1-2-8-28-1-11", name: "수중청소기", spec: "WAVE100", date: "2024.12.30", price: 14000000, account: "남지국민체육센터", photo: "" },
            { id: "120802800100012", assetNo: "1-2-8-28-1-12", name: "수질 및 가스측정 장비", spec: "MR-501SID, PM-2, CSIR-10", date: "2024.04.11", price: 2200000, account: "남지국민체육센터", photo: "" },
            { id: "120802800100013", assetNo: "1-2-8-28-1-13", name: "안전감시대", spec: "SM9000", date: "2024.04.02", price: 1980000, account: "남지국민체육센터", photo: "" },
            { id: "120802800100014", assetNo: "1-2-8-28-1-14", name: "운동물품", spec: "바벨정리대, 고정바벨 10개 세트", date: "2024.01.24", price: 1357000, account: "남지국민체육센터", photo: "" },
            { id: "120802800100015", assetNo: "1-2-8-28-1-15", name: "운영물품(파티션)", spec: "800*60*1170", date: "2024.01.25", price: 185000, account: "남지국민체육센터", photo: "" },
            { id: "120802800100016", assetNo: "1-2-8-28-1-16", name: "운영물품(ㄱ코너)", spec: "60*60*1160", date: "2024.01.25", price: 33000, account: "남지국민체육센터", photo: "" },
            { id: "120802800100017", assetNo: "1-2-8-28-1-17", name: "운영물품(책상)", spec: "1800*700*720", date: "2024.01.25", price: 280000, account: "남지국민체육센터", photo: "" },
            { id: "120802800100018", assetNo: "1-2-8-28-1-18", name: "운영물품(책상의자내자)", spec: "600*630*1160", date: "2024.01.25", price: 500000, account: "남지국민체육센터", photo: "" },
            { id: "120802800100019", assetNo: "1-2-8-28-1-19", name: "운영물품(테이블)", spec: "2100*900*720", date: "2024.01.25", price: 340000, account: "남지국민체육센터", photo: "" },
            { id: "120802800100020", assetNo: "1-2-8-28-1-20", name: "운영물품(테이블 의자내자)", spec: "595*555*835", date: "2024.01.25", price: 840000, account: "남지국민체육센터", photo: "" },
            { id: "120802800100021", assetNo: "1-2-8-28-1-21", name: "운영물품(필경대 타원)", spec: "1350*800*930", date: "2024.01.25", price: 1100000, account: "남지국민체육센터", photo: "" },
            { id: "120802800100022", assetNo: "1-2-8-28-1-22", name: "운영물품(신발장 6단 오픈)", spec: "1200*300*1400", date: "2024.01.25", price: 260000, account: "남지국민체육센터", photo: "" },
            { id: "120802800100023", assetNo: "1-2-8-28-1-23", name: "이동서랍", spec: "400*560*600", date: "2024.08.27", price: 158850, account: "남지국민체육센터", photo: "" },
            { id: "120802800100024", assetNo: "1-2-8-28-1-24", name: "청소도구함", spec: "610*450*1780", date: "2024.04.24", price: 1326600, account: "남지국민체육센터", photo: "" },
            { id: "120802800100025", assetNo: "1-2-8-28-1-25", name: "캐비넷", spec: "800*400*1200", date: "2024.08.27", price: 844530, account: "남지국민체육센터", photo: "" },
            { id: "120802800100026", assetNo: "1-2-8-28-1-26", name: "탈의실 의자", spec: "1100*450*420, 400*400*420", date: "2024.04.26", price: 1621000, account: "남지국민체육센터", photo: "" },
            { id: "120802800100027", assetNo: "1-2-8-28-1-27", name: "헬스장 개인사물함", spec: "310*360*475", date: "2024.08.13", price: 3762000, account: "남지국민체육센터", photo: "" },
            { id: "120802800100028", assetNo: "1-2-8-28-1-28", name: "헬스장 운동기구", spec: "MWW23, MSW05, MSW26", date: "2024.04.18", price: 9171900, account: "남지국민체육센터", photo: "" },
            { id: "120802800100029", assetNo: "1-2-8-28-1-29", name: "혈압측정기", spec: "489*409*284", date: "2024.04.05", price: 1560000, account: "남지국민체육센터", photo: "" },
            { id: "120802800100030", assetNo: "1-2-8-28-1-30", name: "화장실 손건조기", spec: "300*566*197", date: "2024.03.29", price: 2060000, account: "남지국민체육센터", photo: "" },
            { id: "120802800100031", assetNo: "1-2-8-28-1-31", name: "배용 스타트대", spec: "50cm, FINIS", date: "2025.01.22", price: 2200000, account: "남지국민체육센터", photo: "" },
            { id: "120802800100032", assetNo: "1-2-8-28-1-32", name: "기계실 물품관리용 선반", spec: "중량랙, 경량랙, 부품상자", date: "2025.03.05", price: 4202000, account: "남지국민체육센터", photo: "" },
            { id: "120802800100033", assetNo: "1-2-8-28-1-33", name: "헬스장 파워레그 프레스용 원판", spec: "15kg*8개, 20kg*8개, 레그프레스용", date: "2025.04.15", price: 1584000, account: "남지국민체육센터", photo: "" },
            { id: "120802800100034", assetNo: "1-2-8-28-1-34", name: "헬스장 파워레그 프레스", spec: "MSWO2, 2450*1450*1320", date: "2025.04.18", price: 2872350, account: "남지국민체육센터", photo: "" },
            { id: "120802800100035", assetNo: "1-2-8-28-1-35", name: "안내실 사무용 의자", spec: "640*600*1150", date: "2025.12.22", price: 1200000, account: "남지국민체육센터", photo: "" }
        ];

        let html5QrCode = null;
        let currentScannedBarcode = null;

        // 앱 초기화 로직
        window.addEventListener('DOMContentLoaded', () => {
            lucide.createIcons();
            if (!localStorage.getItem('namji_assets')) {
                localStorage.setItem('namji_assets', JSON.stringify(INITIAL_ASSETS));
            }
            renderAssetTable();
        });

        // 탭 전환 함수
        function switchTab(tabName) {
            const tabScanner = document.getElementById('tab-scanner');
            const tabList = document.getElementById('tab-list');
            const btnScanner = document.getElementById('btn-tab-scanner');
            const btnList = document.getElementById('btn-tab-list');

            if (tabName === 'scanner') {
                tabScanner.classList.remove('hidden');
                tabList.classList.add('hidden');
                btnScanner.className = "px-3 py-1.5 rounded-lg text-sm font-medium bg-blue-700 transition flex items-center space-x-1";
                btnList.className = "px-3 py-1.5 rounded-lg text-sm font-medium hover:bg-blue-700 transition flex items-center space-x-1";
            } else {
                tabScanner.classList.add('hidden');
                tabList.classList.remove('hidden');
                btnList.className = "px-3 py-1.5 rounded-lg text-sm font-medium bg-blue-700 transition flex items-center space-x-1";
                btnScanner.className = "px-3 py-1.5 rounded-lg text-sm font-medium hover:bg-blue-700 transition flex items-center space-x-1";
                stopScanner();
                renderAssetTable();
            }
        }

        // 로컬스토리지 데이터 로드
        function getAssets() {
            return JSON.parse(localStorage.getItem('namji_assets')) || [];
        }

        // 바코드 조회 공통 함수
        function searchBarcode(code) {
            if (!code) return;
            code = code.trim();
            currentScannedBarcode = code;
            const assets = getAssets();
            const found = assets.find(item => item.id === code);

            const container = document.getElementById('search-result-container');
            container.classList.remove('hidden');

            if (found) {
                document.getElementById('result-badge').className = "px-2.5 py-1 rounded-full text-xs font-semibold bg-emerald-100 text-emerald-700";
                document.getElementById('result-badge').innerText = "등록된 자산 (확인 완료)";
                document.getElementById('res-asset-name').innerText = found.name;
                document.getElementById('res-barcode').innerText = `바코드 번호: ${found.id}`;
                document.getElementById('res-asset-no').innerText = found.assetNo || '-';
                document.getElementById('res-account-name').innerText = found.account || '-';
                document.getElementById('res-spec').innerText = found.spec || '-';
                document.getElementById('res-date').innerText = found.date || '-';
                document.getElementById('res-price').innerText = found.price ? Number(found.price).toLocaleString() + '원' : '-';

                const photoEl = document.getElementById('res-photo');
                const noPhotoEl = document.getElementById('res-no-photo');
                if (found.photo) {
                    photoEl.src = found.photo;
                    photoEl.classList.remove('hidden');
                    noPhotoEl.classList.add('hidden');
                } else {
                    photoEl.classList.add('hidden');
                    noPhotoEl.classList.remove('hidden');
                }
            } else {
                // 미등록 바코드인 경우 자동 등록 모달 유도
                document.getElementById('result-badge').className = "px-2.5 py-1 rounded-full text-xs font-semibold bg-amber-100 text-amber-700";
                document.getElementById('result-badge').innerText = "미등록 바코드";
                document.getElementById('res-asset-name').innerText = "등록되지 않은 바코드 번호입니다.";
                document.getElementById('res-barcode').innerText = `입력된 번호: ${code}`;
                document.getElementById('res-asset-no').innerText = '-';
                document.getElementById('res-account-name').innerText = '-';
                document.getElementById('res-spec').innerText = '-';
                document.getElementById('res-date').innerText = '-';
                document.getElementById('res-price').innerText = '-';
                document.getElementById('res-photo').classList.add('hidden');
                document.getElementById('res-no-photo').classList.remove('hidden');

                if (confirm(`[${code}]는 미등록 바코드 번호입니다. 지금 바로 신규 등록하시겠습니까?`)) {
                    openAddModalWithBarcode(code);
                }
            }
        }

        // 실시간 카메라 바코드 스캔 시작
        function startScanner() {
            document.getElementById('scanner-placeholder').classList.add('hidden');
            document.getElementById('stop-scanner-btn').classList.remove('hidden');

            html5QrCode = new Html5Qrcode("reader");
            html5QrCode.start(
                { facingMode: "environment" },
                { fps: 10, qrbox: { width: 250, height: 150 } },
                (decodedText) => {
                    document.getElementById('manual-barcode').value = decodedText;
                    searchBarcode(decodedText);
                    playBeep();
                    stopScanner();
                },
                (errorMessage) => {}
            ).catch(err => {
                alert("카메라를 시작할 수 없습니다. 권한 설정을 확인해 주세요.");
                stopScanner();
            });
        }

        // 카메라 스캔 중지
        function stopScanner() {
            if (html5QrCode && html5QrCode.isScanning) {
                html5QrCode.stop().then(() => {
                    html5QrCode.clear();
                    document.getElementById('scanner-placeholder').classList.remove('hidden');
                    document.getElementById('stop-scanner-btn').classList.add('hidden');
                }).catch(err => console.log(err));
            } else {
                document.getElementById('scanner-placeholder').classList.remove('hidden');
                document.getElementById('stop-scanner-btn').classList.add('hidden');
            }
        }

        // 파일 업로드 바코드 스캔
        function scanFileBarcode(event) {
            if (event.target.files.length === 0) return;
            const file = event.target.files[0];
            const html5QrCodeFile = new Html5Qrcode("reader");
            html5QrCodeFile.scanFile(file, true)
                .then(decodedText => {
                    document.getElementById('manual-barcode').value = decodedText;
                    searchBarcode(decodedText);
                    alert(`바코드 인식 성공: ${decodedText}`);
                })
                .catch(err => {
                    alert("이미지에서 바코드를 읽지 못했습니다. 선명한 바코드 이미지를 업로드해 주세요.");
                });
        }

        // 비프음 효과
        function playBeep() {
            const ctx = new (window.AudioContext || window.webkitAudioContext)();
            const osc = ctx.createOscillator();
            const gain = ctx.createGain();
            osc.connect(gain);
            gain.connect(ctx.destination);
            osc.frequency.value = 800;
            gain.gain.setValueAtTime(0.1, ctx.currentTime);
            osc.start();
            osc.stop(ctx.currentTime + 0.1);
        }

        // 자산 목록 테이블 렌더링
        function renderAssetTable() {
            const keyword = document.getElementById('search-keyword').value.toLowerCase();
            const assets = getAssets();
            const tbody = document.getElementById('asset-table-body');
            tbody.innerHTML = '';

            const filtered = assets.filter(item => 
                item.name.toLowerCase().includes(keyword) || 
                item.id.includes(keyword) || 
                (item.assetNo && item.assetNo.toLowerCase().includes(keyword))
            );

            if (filtered.length === 0) {
                tbody.innerHTML = `<tr><td colspan="8" class="text-center py-8 text-slate-400">검색된 자산이 없습니다.</td></tr>`;
                return;
            }

            filtered.forEach(item => {
                const tr = document.createElement('tr');
                tr.className = "hover:bg-slate-50 transition border-b";
                tr.innerHTML = `
                    <td class="p-3">
                        <div class="w-10 h-10 bg-slate-100 rounded-lg overflow-hidden flex items-center justify-center border">
                            ${item.photo ? `<img src="${item.photo}" class="w-full h-full object-cover">` : `<i data-lucide="image" class="w-4 h-4 text-slate-300"></i>`}
                        </div>
                    </td>
                    <td class="p-3 font-mono text-xs font-semibold text-blue-600">${item.id}</td>
                    <td class="p-3 font-medium text-slate-900">${item.name}</td>
                    <td class="p-3 text-xs text-slate-500">${item.assetNo || '-'}</td>
                    <td class="p-3 text-xs text-slate-500">${item.spec || '-'}</td>
                    <td class="p-3 text-xs text-slate-500">${item.date || '-'}</td>
                    <td class="p-3 text-xs font-semibold text-slate-700">${item.price ? Number(item.price).toLocaleString() + '원' : '-'}</td>
                    <td class="p-3 text-center space-x-1">
                        <button onclick="openEditModal('${item.id}')" class="p-1 bg-blue-50 text-blue-600 hover:bg-blue-100 rounded-lg text-xs" title="수정"><i data-lucide="edit" class="w-3.5 h-3.5"></i></button>
                        <button onclick="deleteAsset('${item.id}')" class="p-1 bg-red-50 text-red-600 hover:bg-red-100 rounded-lg text-xs" title="삭제"><i data-lucide="trash-2" class="w-3.5 h-3.5"></i></button>
                    </td>
                `;
                tbody.appendChild(tr);
            });
            lucide.createIcons();
        }

        // 모달 관리
        function openAddModal() {
            document.getElementById('modal-title').innerText = "신규 자산 등록";
            document.getElementById('edit-original-id').value = "";
            document.getElementById('asset-form').reset();
            document.getElementById('modal-photo-preview').classList.add('hidden');
            document.getElementById('modal-no-photo-text').classList.remove('hidden');
            document.getElementById('modal-photo-base64').value = "";
            document.getElementById('asset-modal').classList.remove('hidden');
            document.getElementById('asset-modal').classList.add('flex');
        }

        function openAddModalWithBarcode(code) {
            openAddModal();
            document.getElementById('modal-barcode').value = code;
        }

        function openEditModal(id) {
            const assets = getAssets();
            const item = assets.find(a => a.id === id);
            if (!item) return;

            document.getElementById('modal-title').innerText = "자산 정보 수정";
            document.getElementById('edit-original-id').value = item.id;
            document.getElementById('modal-barcode').value = item.id;
            document.getElementById('modal-name').value = item.name;
            document.getElementById('modal-asset-no').value = item.assetNo || '';
            document.getElementById('modal-spec').value = item.spec || '';
            document.getElementById('modal-account').value = item.account || '';
            document.getElementById('modal-date').value = item.date || '';
            document.getElementById('modal-price').value = item.price || '';

            const preview = document.getElementById('modal-photo-preview');
            const noText = document.getElementById('modal-no-photo-text');
            const base64Input = document.getElementById('modal-photo-base64');

            if (item.photo) {
                preview.src = item.photo;
                preview.classList.remove('hidden');
                noText.classList.add('hidden');
                base64Input.value = item.photo;
            } else {
                preview.classList.add('hidden');
                noText.classList.remove('hidden');
                base64Input.value = "";
            }

            document.getElementById('asset-modal').classList.remove('hidden');
            document.getElementById('asset-modal').classList.add('flex');
        }

        function openEditModalFromScan() {
            if (currentScannedBarcode) {
                openEditModal(currentScannedBarcode);
            }
        }

        function closeModal() {
            document.getElementById('asset-modal').classList.add('hidden');
            document.getElementById('asset-modal').classList.remove('flex');
        }

        // 실물 사진 압축 (Canvas 이용 800px 이하 압축)
        function previewImage(event) {
            const file = event.target.files[0];
            if (!file) return;

            const reader = new FileReader();
            reader.onload = function(e) {
                const img = new Image();
                img.onload = function() {
                    const canvas = document.createElement('canvas');
                    let width = img.width;
                    let height = img.height;
                    const MAX_WIDTH = 800;
                    const MAX_HEIGHT = 800;

                    if (width > height) {
                        if (width > MAX_WIDTH) {
                            height *= MAX_WIDTH / width;
                            width = MAX_WIDTH;
                        }
                    } else {
                        if (height > MAX_HEIGHT) {
                            width *= MAX_HEIGHT / height;
                            height = MAX_HEIGHT;
                        }
                    }

                    canvas.width = width;
                    canvas.height = height;
                    const ctx = canvas.getContext('2d');
                    ctx.drawImage(img, 0, 0, width, height);

                    const dataUrl = canvas.toDataURL('image/jpeg', 0.7); // 품질 0.7 압축
                    document.getElementById('modal-photo-preview').src = dataUrl;
                    document.getElementById('modal-photo-preview').classList.remove('hidden');
                    document.getElementById('modal-no-photo-text').classList.add('hidden');
                    document.getElementById('modal-photo-base64').value = dataUrl;
                }
                img.src = e.target.result;
            }
            reader.readAsDataURL(file);
        }

        // 자산 저장 (추가/수정)
        function saveAsset(event) {
            event.preventDefault();
            const originalId = document.getElementById('edit-original-id').value;
            const newId = document.getElementById('modal-barcode').value.trim();
            
            const newItem = {
                id: newId,
                assetNo: document.getElementById('modal-asset-no').value,
                name: document.getElementById('modal-name').value,
                spec: document.getElementById('modal-spec').value,
                account: document.getElementById('modal-account').value,
                date: document.getElementById('modal-date').value,
                price: Number(document.getElementById('modal-price').value) || 0,
                photo: document.getElementById('modal-photo-base64').value
            };

            let assets = getAssets();
            if (originalId) {
                // 수정
                assets = assets.filter(a => a.id !== originalId);
            }
            // 중복 확인
            if (assets.some(a => a.id === newId)) {
                alert("이미 존재하는 바코드 번호입니다.");
                return;
            }

            assets.unshift(newItem);
            localStorage.setItem('namji_assets', JSON.stringify(assets));
            closeModal();
            renderAssetTable();
            alert("성공적으로 저장되었습니다.");

            if (originalId || currentScannedBarcode === newId) {
                searchBarcode(newId);
            }
        }

        // 자산 삭제
        function deleteAsset(id) {
            if (confirm(`정말 바코드 [${id}] 자산을 삭제하시겠습니까?`)) {
                let assets = getAssets();
                assets = assets.filter(a => a.id !== id);
                localStorage.setItem('namji_assets', JSON.stringify(assets));
                renderAssetTable();
            }
        }

        // CSV 내보내기 (엑셀 한글 깨짐 방지 UTF-8 BOM 적용)
        function exportCSV() {
            const assets = getAssets();
            let csvContent = "\uFEFF바코드번호,자산번호,자산명,규격,취득일자,취득금액,회계명\n";
            assets.forEach(item => {
                csvContent += `"${item.id}","${item.assetNo || ''}","${item.name}","${item.spec || ''}","${item.date || ''}","${item.price || 0}","${item.account || ''}"\n`;
            });

            const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = `남지국민체육센터_자산목록_${new Date().toISOString().slice(0,10)}.csv`;
            a.click();
        }

        // CSV 가져오기
        function importCSV(event) {
            const file = event.target.files[0];
            if (!file) return;

            const reader = new FileReader();
            reader.onload = function(e) {
                const text = e.target.result;
                const lines = text.split('\n');
                let assets = getAssets();
                let count = 0;

                for (let i = 1; i < lines.length; i++) {
                    const line = lines[i].trim();
                    if (!line) continue;
                    const cols = line.split(/,(?=(?:(?:[^"]*"){2})*[^"]*$)/).map(s => s.replace(/^"|"$/g, ''));
                    if (cols.length >= 3) {
                        const [id, assetNo, name, spec, date, price, account] = cols;
                        const existingIdx = assets.findIndex(a => a.id === id);
                        const newItem = {
                            id, assetNo, name, spec, date, 
                            price: Number(price) || 0, 
                            account: account || '남지국민체육센터',
                            photo: existingIdx !== -1 ? assets[existingIdx].photo : ''
                        };
                        if (existingIdx !== -1) {
                            assets[existingIdx] = newItem;
                        } else {
                            assets.push(newItem);
                        }
                        count++;
                    }
                }
                localStorage.setItem('namji_assets', JSON.stringify(assets));
                renderAssetTable();
                alert(`${count}개의 자산 데이터가 성공적으로 병합되었습니다.`);
            };
            reader.readAsText(file, 'utf-8');
        }
    </script>
</body>
</html>
