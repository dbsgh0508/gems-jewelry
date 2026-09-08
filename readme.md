"use client";

import Link from "next/link";
import { useState } from "react";
import PriceChart from "@/components/PriceChart";
import SettingsPanel from "@/components/SettingsPanel";
import type { PriceData } from "@/lib/prices";
import type { Review } from "@/lib/reviews";
import { formatNum } from "@/lib/quote";
import { DON_G, fmtUnit, toUnit, UNIT_LABEL, useSettings } from "@/lib/settings";
import { COMPANY } from "@/lib/company";

export default function Dashboard({
  data,
  reviews,
}: {
  data: PriceData;
  reviews: Review[];
}) {
  const { summary: s, yearly, trust, daily, meta } = data;
  const [st, update, reset] = useSettings();
  const [open, setOpen] = useState(false);

  const last = daily[daily.length - 1];
  const up = s.dayChangePct >= 0;
  const unitLabel = UNIT_LABEL[st.unit];

  const oneDon = Math.round(s.krwG * DON_G);

  // 고객에게 보여주는 공개 참고 가격
  const customerSellPrice =
    Math.round((oneDon * 0.985) / 1000) * 1000;

  const customerBuyPrice =
    Math.round((oneDon * 1.05) / 1000) * 1000;

  return (
    <div className="fade-in">
      <section className="grid-hero" style={{ marginBottom: 20 }}>
        <div>
          <div className="eyebrow">
            Gold Spot · {s.asOf} 종가 기준
          </div>

          <h1
            className="serif h1-hero"
            style={{
              lineHeight: 1.05,
              margin: "8px 0 6px",
              fontWeight: 600,
            }}
          >
            {st.unit === "krwDon"
              ? "순금 1돈"
              : st.unit === "krwG"
              ? "순금 1g"
              : st.unit === "usdOz"
              ? "금 1온스"
              : "달러 환율"}{" "}
            <span className="gold num">
              {fmtUnit(toUnit(last, st.unit), st.unit)}
            </span>
          </h1>

          <div
            style={{
              display: "flex",
              gap: 14,
              flexWrap: "wrap",
              fontSize: 14,
            }}
          >
            <span
              className={
                up ? "verdict-accept" : "verdict-reject"
              }
            >
              전일 {up ? "▲" : "▼"}{" "}
              {formatNum(Math.abs(s.dayChangePct), 2)}%
            </span>

            <span className="muted">
              1돈(3.75g){" "}
              <b
                className="num"
                style={{ color: "var(--text)" }}
              >
                {formatNum(s.krwG * DON_G)}
              </b>
              원
            </span>

            <span className="muted">
              1g{" "}
              <b
                className="num"
                style={{ color: "var(--text)" }}
              >
                {formatNum(s.krwG)}
              </b>
              원
            </span>

            <span className="muted hide-mobile">
              ${formatNum(s.usdOz, 1)}/oz · ₩
              {formatNum(s.krwUsd, 1)}/$
            </span>
          </div>
        </div>

        {st.show.quoteCta && (
          <div
            className="card card-gold"
            style={{ padding: 18 }}
          >
            <div
              className="eyebrow"
              style={{ marginBottom: 8 }}
            >
              내 금, 오늘 얼마?
            </div>

            <p
              style={{
                margin: "0 0 12px",
                fontSize: 14,
                lineHeight: 1.6,
              }}
            >
              사진 한 장으로 품목·순도를 판별하고 오늘
              시세를 기준으로 매입가를 확인합니다.
            </p>

            <div
              style={{
                display: "flex",
                gap: 8,
                flexWrap: "wrap",
              }}
            >
              <Link
                href="/quote"
                className="btn btn-gold"
              >
                매입 견적 받기 →
              </Link>

              <a
                href={COMPANY.store}
                target="_blank"
                rel="noopener noreferrer"
                className="btn btn-store"
              >
                <span className="naver-mark">N</span>
                스마트스토어에서 구매
              </a>
            </div>

            <div
              className="hairline"
              style={{ margin: "14px 0 10px" }}
            />

            <div
              style={{
                display: "flex",
                justifyContent: "space-between",
                alignItems: "center",
                flexWrap: "wrap",
                gap: 8,
                fontSize: 14,
              }}
            >
              <span className="muted">
                전화 상담 · {COMPANY.hours}
              </span>

              <a
                href={COMPANY.phoneHref}
                className="num"
                style={{
                  fontSize: 22,
                  fontWeight: 600,
                  color: "var(--gold2)",
                  letterSpacing: ".02em",
                }}
              >
                ☎ {COMPANY.phone}
              </a>
            </div>
          </div>
        )}
      </section>

      {/* 주요 메뉴 */}
      <section
        className="quick-links"
        aria-label="주요 메뉴"
      >
        <Link
          href="/"
          className="quick-link-card"
        >
          <span className="quick-link-icon">₩</span>

          <span>
            <b>시세</b>
            <small>오늘의 금 시세 확인</small>
          </span>

          <span className="quick-link-arrow">→</span>
        </Link>

        <Link
          href="/quote"
          className="quick-link-card quick-link-gold"
        >
          <span className="quick-link-icon">◈</span>

          <span>
            <b>매입 견적</b>
            <small>사진으로 간편하게 견적</small>
          </span>

          <span className="quick-link-arrow">→</span>
        </Link>

        <Link
          href="/reviews"
          className="quick-link-card"
        >
          <span className="quick-link-icon">★</span>

          <span>
            <b>후기</b>
            <small>실제 고객 후기 확인</small>
          </span>

          <span className="quick-link-arrow">→</span>
        </Link>
      </section>

      {/* 오늘의 금 가격 */}
      <section
        className="price-board"
        aria-label="오늘의 금 가격"
      >
        <div className="price-board-title">
          오늘의 금 가격{" "}
          <span>
            {s.asOf} 기준 · 순금 1돈(3.75g)
          </span>
        </div>

        <div className="price-board-grid">
          <div className="price-board-card sell">
            <div className="price-board-label">
              팔 때 가격
            </div>

            <div className="price-board-value num">
              {formatNum(customerSellPrice)}
              <small>원</small>
            </div>

            <div className="price-board-note">
              오늘 시세 기준 참고가
            </div>
          </div>

          <div className="price-board-card buy">
            <div className="price-board-label">
              살 때 가격
            </div>

            <div className="price-board-value num">
              {formatNum(customerBuyPrice)}
              <small>원</small>
            </div>

            <div className="price-board-note">
              오늘 시세 기준 참고가
            </div>
          </div>
        </div>
      </section>

      {/* 가격 차트 */}
      <section
        className="card"
        style={{
          padding: 18,
          marginBottom: 20,
        }}
      >
        <PriceChart
          daily={daily}
          unit={st.unit}
          onUnit={(unit) => update({ unit })}
          range={st.range}
          onRange={(range) => update({ range })}
        />
      </section>

      {/* 기간별 등락 */}
      {st.show.tiles && (
        <section
          className="grid-4"
          style={{ marginBottom: 20 }}
        >
          {(
            [
              ["1개월", s.changeVs["1M"]],
              ["1년", s.changeVs["1Y"]],
              ["5년", s.changeVs["5Y"]],
              ["2015년 이후", s.changeVs.since2015],
            ] as const
          ).map(([label, v]) => (
            <div
              key={label}
              className="card"
              style={{ padding: "14px 16px" }}
            >
              <div
                className="muted"
                style={{ fontSize: 12 }}
              >
                {label}
              </div>

              <div
                className={`num ${
                  v >= 0
                    ? "verdict-accept"
                    : "verdict-reject"
                }`}
                style={{
                  fontSize: 26,
                  fontWeight: 600,
                  marginTop: 2,
                }}
              >
                {v >= 0 ? "+" : ""}
                {formatNum(v, 1)}%
              </div>
            </div>
          ))}
        </section>
      )}

      <section
        className="grid-2"
        style={{ marginBottom: 20 }}
      >
        {st.show.yearly && (
          <div
            className="card"
            style={{ padding: 18 }}
          >
            <h3
              className="serif"
              style={{
                fontSize: 22,
                margin: "0 0 10px",
                fontWeight: 600,
              }}
            >
              연도별 등락{" "}
              <span
                className="muted"
                style={{
                  fontSize: 12,
                  fontFamily: "var(--font-body)",
                }}
              >
                (
                {unitLabel === "₩ / $"
                  ? "원 / g"
                  : unitLabel}
                )
              </span>
            </h3>

            <div style={{ overflowX: "auto" }}>
              <table
                style={{
                  width: "100%",
                  fontSize: 14,
                  borderCollapse: "collapse",
                  minWidth: 360,
                }}
              >
                <thead>
                  <tr
                    className="muted"
                    style={{
                      fontSize: 12,
                      textAlign: "right",
                    }}
                  >
                    <th
                      style={{
                        textAlign: "left",
                        padding: "6px 0",
                      }}
                    >
                      연도
                    </th>
                    <th>연말</th>
                    <th>최고</th>
                    <th>원화</th>
                    <th>달러</th>
                  </tr>
                </thead>

                <tbody>
                  {[...yearly]
                    .reverse()
                    .map((y) => {
                      const k =
                        st.unit === "krwDon"
                          ? DON_G
                          : 1;

                      return (
                        <tr
                          key={y.year}
                          style={{
                            borderTop:
                              "1px solid var(--line)",
                            textAlign: "right",
                          }}
                        >
                          <td
                            style={{
                              textAlign: "left",
                              padding: "7px 0",
                            }}
                            className="num"
                          >
                            {y.year}
                          </td>

                          <td className="num">
                            {formatNum(y.close * k)}
                          </td>

                          <td className="num muted">
                            {formatNum(y.high * k)}
                          </td>

                          <td
                            className={`num ${
                              y.changePct >= 0
                                ? "verdict-accept"
                                : "verdict-reject"
                            }`}
                          >
                            {y.changePct >= 0
                              ? "+"
                              : ""}
                            {formatNum(
                              y.changePct,
                              1
                            )}
                            %
                          </td>

                          <td
                            className={`num ${
                              y.changeUsdPct >= 0
                                ? "verdict-accept"
                                : "verdict-reject"
                            }`}
                          >
                            {y.changeUsdPct >= 0
                              ? "+"
                              : ""}
                            {formatNum(
                              y.changeUsdPct,
                              1
                            )}
                            %
                          </td>
                        </tr>
                      );
                    })}
                </tbody>
              </table>
            </div>
          </div>
        )}

        <div
          style={{
            display: "flex",
            flexDirection: "column",
            gap: 14,
          }}
        >
          {st.show.reviews && (
            <div
              className="card"
              style={{
                padding: 18,
                flex: 1,
              }}
            >
              <div
                style={{
                  display: "flex",
                  justifyContent: "space-between",
                  alignItems: "baseline",
                }}
              >
                <h3
                  className="serif"
                  style={{
                    fontSize: 22,
                    margin: 0,
                    fontWeight: 600,
                  }}
                >
                  최근 후기
                </h3>

                <Link
                  href="/reviews"
                  className="muted"
                  style={{ fontSize: 13 }}
                >
                  전체 보기 →
                </Link>
              </div>

              {reviews.map((r) => (
                <div
                  key={r.id}
                  style={{
                    borderTop:
                      "1px solid var(--line)",
                    padding: "10px 0",
                  }}
                >
                  <div
                    style={{
                      display: "flex",
                      justifyContent:
                        "space-between",
                      fontSize: 13,
                    }}
                  >
                    <span>
                      <b>{r.name}</b>{" "}
                      <span className="muted">
                        · {r.item}
                      </span>
                    </span>

                    <span className="star">
                      {"★".repeat(r.rating)}
                      <span className="star-off">
                        {"★".repeat(
                          5 - r.rating
                        )}
                      </span>
                    </span>
                  </div>

                  <p
                    className="muted"
                    style={{
                      margin: "5px 0 0",
                      fontSize: 14,
                      lineHeight: 1.6,
                    }}
                  >
                    {r.text}
                  </p>
                </div>
              ))}
            </div>
          )}

          {st.show.trust && (
            <div
              className="card"
              style={{
                padding: "14px 18px",
                fontSize: 13,
                lineHeight: 1.7,
              }}
            >
              <div
                className="eyebrow"
                style={{ marginBottom: 6 }}
              >
                데이터 신뢰도
              </div>

              LBMA 월별 공식가와{" "}
              {trust.lbmaMonthsCompared}개월 대조 —
              평균 오차{" "}
              <b className="num">
                {formatNum(
                  trust.meanAbsDiffPct,
                  2
                )}
                %
              </b>
              , 최대{" "}
              {formatNum(
                trust.maxAbsDiffPct,
                2
              )}
              %.

              <span className="muted">
                {" "}
                출처:{" "}
                {meta.sources
                  .map((x) => x.via)
                  .filter(
                    (v, i, a) =>
                      a.indexOf(v) === i
                  )
                  .join(", ")}
                .
              </span>
            </div>
          )}
        </div>
      </section>

      <SettingsPanel
        open={open}
        onOpen={() => setOpen(true)}
        onClose={() => setOpen(false)}
        s={st}
        update={update}
        reset={reset}
      />
    </div>
  );
}