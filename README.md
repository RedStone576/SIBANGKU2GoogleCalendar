# Convert Jadwal di SIBANGKU ke .ics Untuk Import Google Calendar

## Cara Menggunakan

1. Login ke [SIBANGKU](https://ujian.cs.ui.ac.id/sibangku/) lalu navigasi ke page [**Jadwal**](https://ujian.cs.ui.ac.id/sibangku/jadwal/)
2. Buka devtool dengan `F12` atau (`ctrl` + `shift` + `i`)
3. Copy dan paste script berikut di Console; Akan ter-download file .ics yang dapat di-import ke Google Calendar
```js
;(() =>
{
    const M={Jan:0,Feb:1,Mar:2,Apr:3,May:4,Jun:5,Jul:6,Aug:7,Sep:8,Oct:9,Nov:10,Dec:11}
    const P=n=>String(n).padStart(2,'0')
    const F=d=>d.getFullYear()+P(d.getMonth()+1)+P(d.getDate())+'T'+P(d.getHours())+P(d.getMinutes())+'00'

    const events=[...document.querySelectorAll('.card.mb-4')].map(card=>
    {
        const G=l=>[...card.querySelectorAll('.card-body>div')]
            .find(d=>d.textContent.includes(l))
            ?.querySelector('strong')?.textContent.trim()

        const seat=G('Nomor Bangku')
        const [day,month,year]=G('Hari/Tanggal').split(', ')[1].split('-')
        const [s,e]=G('Waktu').split(' - ')

        const D=t=>
        {
            const [h,m]=t.split('.').map(Number)
            return new Date(2000+ +year,M[month],+day,h,m)
        }

        return {
            summary:`${card.querySelector('.card-header strong').textContent.trim()} - Bangku ${seat}`,
            location:G('Ruangan'),
            description:`Nomor Bangku: ${seat}`,
            start:F(D(s)),
            end:F(D(e))
        }
    })

    const ics=[
        'BEGIN:VCALENDAR',
        'VERSION:2.0',
        'PRODID:-//Schedule Export//EN',

        ...events.flatMap(e=>[
            'BEGIN:VEVENT',
            `SUMMARY:${e.summary}`,
            `DTSTART:${e.start}`,
            `DTEND:${e.end}`,
            `LOCATION:${e.location}`,
            `DESCRIPTION:${e.description}`,
            'END:VEVENT'
        ]),

        'END:VCALENDAR'
    ].join('\r\n')

    const blob=new Blob([ics],{type:'text/calendar'})
    const a=document.createElement('a')

    a.href=URL.createObjectURL(blob)
    a.download='schedule.ics'
    a.click()
})();
```
