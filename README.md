[Tugas 7 IAMS -Risfadilla Dwi Wulandari.json](https://github.com/user-attachments/files/27964137/Tugas.7.IAMS.-Risfadilla.Dwi.Wulandari.json)
{
  "name": "Tugas 7 IAMS -Risfadilla Dwi Wulandari",
  "nodes": [
    {
      "parameters": {
        "formTitle": "Form Rekrutmen Posisi Manajemen Marketing",
        "formDescription": "Harap isi form di bawah ini sesuai dnegan ketentuan yang berlaku.",
        "formFields": {
          "values": [
            {
              "fieldLabel": "Nama Lengkap",
              "placeholder": "Budi Santoso",
              "requiredField": true
            },
            {
              "fieldLabel": "Pengalaman",
              "fieldType": "dropdown",
              "fieldOptions": {
                "values": [
                  {
                    "option": "< 1 Tahun"
                  },
                  {
                    "option": "1-3 Tahun"
                  },
                  {
                    "option": "> 3 Tahun"
                  }
                ]
              },
              "requiredField": true
            },
            {
              "fieldLabel": "Upload CV (PDF)",
              "fieldType": "file",
              "acceptFileTypes": "pdf",
              "requiredField": true
            }
          ]
        },
        "options": {}
      },
      "type": "n8n-nodes-base.formTrigger",
      "typeVersion": 2.5,
      "position": [
        0,
        0
      ],
      "id": "a08f6601-e278-40df-b5d0-edfaaef91d63",
      "name": "On form submission",
      "webhookId": "be05d749-bf87-40ce-a699-4076712f4e25"
    },
    {
      "parameters": {
        "jsCode": "const items = $input.all(); \n\nfor (let item of items) {\n  const binaryData = item.binary?.Upload_CV__PDF_;\n\n  if (binaryData) {\n    item.json.cv_base64 = binaryData.data; \n    item.json.fileName = binaryData.fileName;\n  }\n\n  item.json.tanggal_input = DateTime.now().toFormat('dd-MM-yyyy HH:mm');\n}\n\nreturn items;\n"
      },
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [
        208,
        0
      ],
      "id": "602ff45a-f859-46e1-a7c3-1b1790cdfbe7",
      "name": "Code in JavaScript"
    },
    {
      "parameters": {
        "operation": "pdf",
        "binaryPropertyName": "Upload_CV__PDF_",
        "options": {}
      },
      "type": "n8n-nodes-base.extractFromFile",
      "typeVersion": 1.1,
      "position": [
        416,
        0
      ],
      "id": "0c1036b3-a9e4-4b75-a403-c518967b7dd9",
      "name": "Extract from File"
    },
    {
      "parameters": {
        "promptType": "define",
        "text": "=Peran: Anda adalah Senior Technical Recruiter yang sangat teliti dan objektif. Tugas Anda adalah mengevaluasi teks CV kandidat di bawah ini untuk posisi Automation Engineer/Developer.\n\nKonteks CV Kandidat:\n\"\"\"\n{{ $json.text }}\n\"\"\"\n\nInstruksi Penilaian:\n1. Analisis pengalaman kerja, keahlian teknis (Hard Skills), dan latar belakang pendidikan.\n2. Identifikasi apakah kandidat memiliki pengalaman dengan tool seperti: n8n, Javascript, Python, SQL, atau API Integration.\n3. Berikan skor profesional dari 0-100.\n   - Skor < 50: Tidak relevan/Pemula.\n   - Skor 50-75: Menengah/Potensial.\n   - Skor > 75: Sangat Cocok/Senior.\n\nFormat Output (WAJIB JSON):\nJangan berikan basa-basi atau teks pembuka. HANYA berikan output JSON valid dengan struktur berikut:\n{\n  \"candidate_name\": \"Nama lengkap kandidat (jika ada)\",\n  \"years_of_experience\": \"Estimasi total tahun pengalaman (angka)\",\n  \"top_skills\": [\"Skill 1\", \"Skill 2\", \"Skill 3\", \"Skill 4\", \"Skill 5\"],\n  \"summary\": \"Ringkasan profil kandidat dalam 1 paragraf pendek (maksimal 30 kata) dalam Bahasa Indonesia.\",\n  \"strengths\": \"Poin kuat kandidat\",\n  \"weaknesses\": \"Kekurangan/hal yang perlu diklarifikasi\",\n  \"score\": 0,\n  \"hiring_recommendation\": \"YES / NO / MAYBE\"\n}\n",
        "batching": {}
      },
      "type": "@n8n/n8n-nodes-langchain.chainLlm",
      "typeVersion": 1.9,
      "position": [
        624,
        0
      ],
      "id": "0e6ad538-2ec5-4499-8ded-82ff59e34f5b",
      "name": "Basic LLM Chain"
    },
    {
      "parameters": {
        "options": {}
      },
      "type": "@n8n/n8n-nodes-langchain.lmChatGoogleGemini",
      "typeVersion": 1,
      "position": [
        784,
        128
      ],
      "id": "1894e395-5c4e-4c90-a5b3-4a7de7a6bdff",
      "name": "Google Gemini Chat Model",
      "credentials": {
        "googlePalmApi": {
          "id": "u1xxYbsbTuSagj2o",
          "name": "Google Gemini(PaLM) Api account"
        }
      }
    },
    {
      "parameters": {
        "jsCode": "const items = $input.all();\n\nfor (let item of items) {\n  const aiResponseText = item.json.text;\n  \n  try {\n\n    const cleanJson = aiResponseText.replace(/```json|```/g, '').trim();\n    \n    const parsedData = JSON.parse(cleanJson);\n    \n    item.json.ai_name = parsedData.candidate_name;\n    item.json.ai_exp = parsedData.years_of_experience;\n    item.json.ai_skills = parsedData.top_skills.join(\", \"); // Array jadi string\n    item.json.ai_summary = parsedData.summary;\n    item.json.ai_score = parsedData.score;\n    item.json.ai_rec = parsedData.hiring_recommendation;\n    \n  } catch (error) {\n    item.json.error = \"Gagal parsing JSON dari AI\";\n    item.json.raw_output = aiResponseText;\n  }\n}\n\nreturn items;"
      },
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [
        976,
        0
      ],
      "id": "b3492b38-abdc-49ea-83e8-e932e59ca45c",
      "name": "Code in JavaScript1"
    },
    {
      "parameters": {
        "chatId": "1181462723",
        "text": "=Nama: {{ $json.ai_name }}\nSkills: {{ $json.ai_skills }}\nScore: {{ $json.ai_score }}\nSummary:{{ $json.ai_summary }}",
        "additionalFields": {}
      },
      "type": "n8n-nodes-base.telegram",
      "typeVersion": 1.2,
      "position": [
        1184,
        0
      ],
      "id": "29d41704-0604-4183-a044-a518d82dc307",
      "name": "Send a text message",
      "webhookId": "93756205-9137-4cff-9f16-56b306963c93",
      "credentials": {
        "telegramApi": {
          "id": "rZnWxdav23m89bvT",
          "name": "Telegram account"
        }
      }
    }
  ],
  "pinData": {},
  "connections": {
    "On form submission": {
      "main": [
        [
          {
            "node": "Code in JavaScript",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Code in JavaScript": {
      "main": [
        [
          {
            "node": "Extract from File",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Extract from File": {
      "main": [
        [
          {
            "node": "Basic LLM Chain",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Google Gemini Chat Model": {
      "ai_languageModel": [
        [
          {
            "node": "Basic LLM Chain",
            "type": "ai_languageModel",
            "index": 0
          }
        ]
      ]
    },
    "Basic LLM Chain": {
      "main": [
        [
          {
            "node": "Code in JavaScript1",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Code in JavaScript1": {
      "main": [
        [
          {
            "node": "Send a text message",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  },
  "active": false,
  "settings": {
    "executionOrder": "v1",
    "binaryMode": "separate"
  },
  "versionId": "8c4158d3-9e38-4ae1-bb29-b6fc3cdb056d",
  "meta": {
    "templateCredsSetupCompleted": true,
    "instanceId": "a37b3d99a758656b91fe65d1c1a66592c8e864a6b62f23344415d201880e7dfc"
  },
  "id": "4YYvTB6fA7TRR7Th",
  "tags": []
}
